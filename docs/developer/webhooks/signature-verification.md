# Signature Verification

Every webhook request from Scholaro is signed. Verifying the signature confirms that the request came from Scholaro and that the body was not changed on the way. It is optional, but we recommend it.

## How the signature is made

Each request includes an `X-Scholaro-Signature` header. Its value is:

```text
Base64( HMAC-SHA256( request body, signing secret ) )
```

- **Request body:** the raw request body, exactly as received, as UTF-8 bytes.
- **Signing secret:** the **Signing Secret** shown for your endpoint in the [Developer dashboard](https://www.scholaro.com/app/developer/webhooks). Each endpoint has its own secret.

## Verify a request

1. **Read the raw body.** Read the request body as bytes, before any JSON parsing. Parsing and re-serializing the JSON can change spacing or number formatting, and the signature will no longer match.
2. **Get the header.** Read the `X-Scholaro-Signature` header.
3. **Compute the HMAC.** Compute an HMAC-SHA256 of the body bytes, using the UTF-8 bytes of your signing secret as the key.
4. **Encode it.** Encode the result as a Base64 string.
5. **Compare.** Compare it with the header value using a constant-time comparison. If they don't match, reject the request.

## Examples

=== "C#"

    ```csharp
    using System.Security.Cryptography;
    using System.Text;

    static bool IsValidScholaroSignature(byte[] body, string signatureHeader, string signingSecret)
    {
        using var hmac = new HMACSHA256(Encoding.UTF8.GetBytes(signingSecret));
        byte[] expected = Encoding.UTF8.GetBytes(Convert.ToBase64String(hmac.ComputeHash(body)));
        byte[] received = Encoding.UTF8.GetBytes(signatureHeader ?? "");
        return CryptographicOperations.FixedTimeEquals(expected, received);
    }
    ```

=== "Node.js"

    ```javascript
    const crypto = require("crypto");

    // body must be the raw request body as a Buffer, e.g. from express.raw({ type: "application/json" })
    function isValidScholaroSignature(body, signatureHeader, signingSecret) {
      const expected = Buffer.from(
        crypto.createHmac("sha256", signingSecret).update(body).digest("base64")
      );
      const received = Buffer.from(signatureHeader || "");
      return expected.length === received.length && crypto.timingSafeEqual(expected, received);
    }
    ```

=== "Python"

    ```python
    import base64
    import hashlib
    import hmac

    def is_valid_scholaro_signature(body: bytes, signature_header: str, signing_secret: str) -> bool:
        digest = hmac.new(signing_secret.encode("utf-8"), body, hashlib.sha256).digest()
        expected = base64.b64encode(digest).decode("ascii")
        return hmac.compare_digest(expected, signature_header or "")
    ```

!!! warning "Keep your signing secret private"
    Anyone with the signing secret can create requests that pass verification. Store it like a password, and don't commit it to source control.

## Related docs

- [Webhooks Overview](overview.md)
- [Events Reference](events-reference.md)
