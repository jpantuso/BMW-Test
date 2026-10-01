# 03. Swift sample: a BMW CarData client

A dependency-free Swift client for the CarData customer API: PKCE, device-code login, token refresh, and the REST calls. It type-checks against the macOS 14 SDK (`swiftc -typecheck`, 2026-10-01). It has **not** been run against BMW, since that needs a CarData Client ID from an EU/UK account (see [Questions](../Questions.md) Q1). Endpoint details and limits are in [01](01-current-state.md#bmw-cardata-technically).

CarData is read-only, so there is nothing here that commands the car. Commands go through the bridge in [`integration-spec.md`](../integration-spec.md).

## Usage

```swift
let client = CarDataClient(clientID: "<client ID from the CarData portal>")
let pkce = CarDataClient.PKCE()
let code = try await client.requestDeviceCode(pkce: pkce)
print("Approve at \(code.verification_uri_complete)")
var tokens = try await client.pollForTokens(code, pkce: pkce)
// Persist tokens in the Keychain. Refresh before the 1-hour access token expires;
// each refresh also renews the 2-week refresh token.
tokens = try await client.refresh(tokens)
let vehicles = try await client.mappings(tokens)
```

The MQTT stream needs an MQTT client (CocoaMQTT or `mqtt-nio`): connect with TLS to `customer.streaming-cardata.bmwgroup.com:9000`, username `tokens.gcid`, password `tokens.id_token`, and subscribe to `<VIN>/#`. Reconnect with a fresh ID token each hour.

## Source

```swift
import CryptoKit
import Foundation

/// Minimal client for BMW CarData (customer API). Read-only: CarData has no command endpoints.
struct CarDataClient {
    static let authBase = URL(string: "https://customer.bmwgroup.com")!
    static let apiBase = URL(string: "https://api-cardata.bmwgroup.com")!

    let clientID: String  // generated in the CarData customer portal
    var session: URLSession = .shared

    struct DeviceCode: Decodable {
        let user_code: String
        let device_code: String
        let interval: Int
        let expires_in: Int
        let verification_uri_complete: String
    }

    struct Tokens: Codable {
        let access_token: String  // REST, 1 hour
        let id_token: String      // MQTT password, 1 hour
        let refresh_token: String // 2 weeks; rerun the device flow after that
        let gcid: String          // MQTT username
        let expires_in: Int
    }

    struct PKCE {
        let verifier: String
        let challenge: String

        init() {
            var bytes = [UInt8](repeating: 0, count: 32)
            _ = SecRandomCopyBytes(kSecRandomDefault, bytes.count, &bytes)
            verifier = Data(bytes).base64URL
            challenge = Data(SHA256.hash(data: Data(verifier.utf8))).base64URL
        }
    }

    enum CarDataError: Error { case http(Int, String) }

    // MARK: Device code flow

    func requestDeviceCode(pkce: PKCE) async throws -> DeviceCode {
        try await postForm("/gcdm/oauth/device/code", [
            "client_id": clientID,
            "response_type": "device_code",
            "scope": "authenticate_user openid cardata:api:read cardata:streaming:read",
            "code_challenge": pkce.challenge,
            "code_challenge_method": "S256",
        ])
    }

    /// Polls until the user has approved the code at `verification_uri_complete`.
    func pollForTokens(_ code: DeviceCode, pkce: PKCE) async throws -> Tokens {
        let deadline = Date().addingTimeInterval(TimeInterval(code.expires_in))
        while Date() < deadline {
            try await Task.sleep(for: .seconds(code.interval))
            do {
                return try await postForm("/gcdm/oauth/token", [
                    "client_id": clientID,
                    "device_code": code.device_code,
                    "grant_type": "urn:ietf:params:oauth:grant-type:device_code",
                    "code_verifier": pkce.verifier,
                ])
            } catch CarDataError.http(_, let body) where body.contains("authorization_pending") {
                continue
            }
        }
        throw CarDataError.http(408, "device code expired")
    }

    func refresh(_ tokens: Tokens) async throws -> Tokens {
        try await postForm("/gcdm/oauth/token", [
            "client_id": clientID,
            "grant_type": "refresh_token",
            "refresh_token": tokens.refresh_token,
        ])
    }

    // MARK: REST (50 calls per day per account; the MQTT stream is the primary feed)

    func mappings(_ tokens: Tokens) async throws -> Data {
        try await get("/customers/vehicles/mappings", tokens)
    }

    /// `containerID` comes from POST /customers/containers, which names the telematic keys you want.
    func telematicData(vin: String, containerID: String, _ tokens: Tokens) async throws -> Data {
        try await get("/customers/vehicles/\(vin)/telematicData?containerId=\(containerID)", tokens)
    }

    // MARK: Plumbing

    private func postForm<T: Decodable>(_ path: String, _ fields: [String: String]) async throws -> T {
        var request = URLRequest(url: Self.authBase.appending(path: path))
        request.httpMethod = "POST"
        request.setValue("application/x-www-form-urlencoded", forHTTPHeaderField: "Content-Type")
        request.setValue("application/json", forHTTPHeaderField: "Accept")
        var form = URLComponents()
        form.queryItems = fields.map { URLQueryItem(name: $0.key, value: $0.value) }
        request.httpBody = Data((form.percentEncodedQuery ?? "").utf8)
        return try JSONDecoder().decode(T.self, from: try await send(request))
    }

    private func get(_ pathAndQuery: String, _ tokens: Tokens) async throws -> Data {
        var request = URLRequest(url: URL(string: pathAndQuery, relativeTo: Self.apiBase)!)
        request.setValue("Bearer \(tokens.access_token)", forHTTPHeaderField: "Authorization")
        request.setValue("v1", forHTTPHeaderField: "x-version")
        request.setValue("application/json", forHTTPHeaderField: "Accept")
        return try await send(request)
    }

    private func send(_ request: URLRequest) async throws -> Data {
        let (data, response) = try await session.data(for: request)
        let status = (response as? HTTPURLResponse)?.statusCode ?? 0
        guard (200..<300).contains(status) else {
            throw CarDataError.http(status, String(decoding: data, as: UTF8.self))
        }
        return data
    }
}

extension Data {
    var base64URL: String {
        base64EncodedString()
            .replacingOccurrences(of: "+", with: "-")
            .replacingOccurrences(of: "/", with: "_")
            .replacingOccurrences(of: "=", with: "")
    }
}
```
