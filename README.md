FIDO 2 = wealths + ctap



1. registration 
   - request from server to browser
   - CTAP to browser - PublicKeyCredential
   - browser to server - JSON(PublicKeyCredentials)
   - store credentials - public key, credential id
2. login



```go
type PublicKeyCredentialCreationOptions	
  Challenge              Challenge                `json:"challenge"`
	RelyingParty           RelyingPartyEntity       `json:"rp"`
	User                   UserEntity               `json:"user"`
```





```go
type PublicKeyCredentialRequestOptions
	Challenge          Challenge                   `json:"challenge"`
```



Challenge - at least 16 random bytes, base64 encoded



```go
type RelyingPartyEntity 
	CredentialEntity
	ID string `json:"id"`
```

ID - origin url, including https



```go
type UserEntity struct {
	CredentialEntity
	ID []byte `json:"id"`
}

```



```go
type CredentialEntity 
	Name string `json:"name"`
```





```go
type CredentialParameter
	Type      CredentialType                       `json:"type"`
	Algorithm webauthncose.COSEAlgorithmIdentifier `json:"alg"`
```



CredentialType - string = 'public-key'

Algorithm - (-7 = ES256, -257 = RS256)

// AlgES256 ECDSA with SHA-256

// AlgRS256 RSASSA-PKCS1-v1_5 with SHA-256



Registration Request - by browser

- get credential
- generate public key
- register



Request: 

GET https://webauthn.io/makeCredential/laurentiu?attType=none&authType=&userVerification=discouraged&residentKeyRequirement=false&txAuthExtension=

Response:

```json
{
  "publicKey": {
    "challenge": "WSx04mhXB4LE+CPmpywDMjzFBC2e+SMWxECdREuhNLw=",
    "rp": {
      "name": "webauthn.io",
      "id": "webauthn.io"
    },
    "user": {
      "name": "laurentiu",
      "displayName": "laurentiu",
      "id": "tfMHAAAAAAAAAA=="
    },
    "pubKeyCredParams": [
      {
        "type": "public-key",
        "alg": -7
      }
    ],
    "authenticatorSelection": {
      "requireResidentKey": false,
      "userVerification": "discouraged"
    },
    "timeout": 60000,
    "extensions": {
      "txAuthSimple": ""
    },
    "attestation": "none"
  }
}
```



```javascript
navigator.credentials.create({
                "publicKey": {
                    "challenge": Uint8Array.from(atob("WSx04mhXB4LE+CPmpywDMjzFBC2e+SMWxECdREuhNLw="), c => c.charCodeAt(0)),
                    "rp": {
                    "name": "localhost",
                    "id": "localhost"
                    },
                    "user": {
                    "name": "laurentiu",
                    "displayName": "laurentiu",
                    "id": Uint8Array.from(atob("tfMHAAAAAAAAAA=="), c => c.charCodeAt(0)),
                    },
                    "pubKeyCredParams": [
                    {
                        "type": "public-key",
                        "alg": -7
                    }
                    ],
                    "authenticatorSelection": {
                        "authenticatorAttachment": "platform",
                        "requireResidentKey": false,
                        "userVerification": "discouraged"
                    },
                    "timeout": 60000,
                    "extensions": {
                    "txAuthSimple": ""
                    },
                    "attestation": "direct"
                }
            }).then(function (newCredential){
                console.log(newCredential)
            })
```



```javascript
navigator.credentials.create({
                "publicKey": {
                    "challenge": Uint8Array.from(atob("WSx04mhXB4LE+CPmpywDMjzFBC2e+SMWxECdREuhNLw="), c => c.charCodeAt(0)),
                    "rp": {
                    "name": "localhost",
                    "id": "localhost"
                    },
                    "user": {
                    "name": "laurentiu",
                    "displayName": "laurentiu",
                    "id": Uint8Array.from(atob("tfMHAAAAAAAAAA=="), c => c.charCodeAt(0)),
                    },
                    "pubKeyCredParams": [
                    {
                        "type": "public-key",
                        "alg": -7
                    }
                    ],
                    "authenticatorSelection": {
                        "authenticatorAttachment": "platform",
                        "requireResidentKey": false,
                        "userVerification": "discouraged"
                    },
                    "timeout": 60000,
                    "extensions": {
                    "txAuthSimple": ""
                    },
                    "attestation": "direct"
                }
            }).then(function (newCredential){
                
                let attestationObject = new Uint8Array(newCredential.response.attestationObject);
                let clientDataJSON = new Uint8Array(newCredential.response.clientDataJSON);
                let rawId = new Uint8Array(newCredential.rawId);

                let jsonRequest = JSON.stringify({
                        id: newCredential.id,
                        rawId: bufferEncode(rawId),
                        type: newCredential.type,
                        response: {
                            attestationObject: bufferEncode(attestationObject),
                            clientDataJSON: bufferEncode(clientDataJSON),
                        }
                })                

                console.log(jsonRequest)
            });

            function bufferEncode(value) {
                return _arrayBufferToBase64(value)
                    .replace(/\+/g, "-")
                    .replace(/\//g, "_")
                    .replace(/=/g, "");
            }

            function _arrayBufferToBase64( buffer ) {
                var binary = '';
                var bytes = new Uint8Array( buffer );
                var len = bytes.byteLength;
                for (var i = 0; i < len; i++) {
                    binary += String.fromCharCode( bytes[ i ] );
                }
                return window.btoa( binary );
            }
```



```json
{
    "id": "AQKreZz6wNc15NVNeOk2mkpkRWgy8_T7vo3cym71Fj3Z7e03CPmpfyG0MjMmNRpHyAwomm1RfbpGRMnL8s4",
    "rawId": "AQKreZz6wNc15NVNeOk2mkpkRWgy8_T7vo3cym71Fj3Z7e03CPmpfyG0MjMmNRpHyAwomm1RfbpGRMnL8s4",
    "type": "public-key",
    "response": {
        "attestationObject": "o2NmbXRmcGFja2VkZ2F0dFN0bXSiY2FsZyZjc2lnWEcwRQIhAKS4ZZBVdQ3nZNsQqw2Jo6FS4xO-I7T1Qn9GAgR58IysAiAz2z5VdUTtifRAlgOUgzhwQm-DODrI782Qn8MoPiXWG2hhdXRoRGF0YVjCSZYN5YgOjGh0NBcPZHZgW4_krrmihjLHmVzzuoMdl2NFXuksMK3OAAI1vMYKZIsLJfHwVQMAPgECq3mc-sDXNeTVTXjpNppKZEVoMvP0-76N3Mpu9RY92e3tNwj5qX8htDIzJjUaR8gMKJptUX26RkTJy_LOpQECAyYgASFYIET6arx7Y5VQD8BsXXdT1eIdSCiXT2c133lkYkR0VYR5IlgggbyDVcxB2SKNoU46XkG459cvPsdRK8H8Ug0mreGy2q0",
        "clientDataJSON": "eyJ0eXBlIjoid2ViYXV0aG4uY3JlYXRlIiwiY2hhbGxlbmdlIjoiV1N4MDRtaFhCNExFLUNQbXB5d0RNanpGQkMyZS1TTVd4RUNkUkV1aE5MdyIsIm9yaWdpbiI6Imh0dHA6Ly9sb2NhbGhvc3Q6NTUwMCIsImNyb3NzT3JpZ2luIjpmYWxzZSwiZXh0cmFfa2V5c19tYXlfYmVfYWRkZWRfaGVyZSI6ImRvIG5vdCBjb21wYXJlIGNsaWVudERhdGFKU09OIGFnYWluc3QgYSB0ZW1wbGF0ZS4gU2VlIGh0dHBzOi8vZ29vLmdsL3lhYlBleCJ9"
    }
}
```

LOGIN

GET https://webauthn.io/user/laurentiu/exists

```json
{
  "exists": true
}
```





GET https://webauthn.io/assertion/laurentiu?userVer=discouraged&txAuthExtension=

```json
{
  "publicKey": {
    "challenge": "0vOmk/FxFHW0hqyN3nH6JTwLWO0xUSkfhWYkCe9SJMo=",
    "timeout": 60000,
    "rpId": "webauthn.io",
    "allowCredentials": [
      {
        "type": "public-key",
        "id": "AUONF17pXN3E5xDCtpchlpyYkgsKykgcBKHWG1CIE0VrMCs3okm6utA3VhQyOnvf9B8GH0ektsVhpDKS/KU="
      },
      {
        "type": "public-key",
        "id": "AcikYqbdAAYYm6fviUIbeEZrB1lI+X6XbrbcRFUm+DUBGsxdmBx9/W4RFVa3Dso+ld5DgRL7TqVTNVWisJE="
      },
      {
        "type": "public-key",
        "id": "AWU5bKbQIJGJfbRAJcuJKUVCKMOmIl7uDhHfIWiy0YBcn1fHz6zCWS91LFhDp4EUg+KntdmprFjfFSJmR9g="
      }
    ],
    "extensions": {
      "txAuthSimple": ""
    }
  }
}
```

=> navigator.credentials.get()

POST https://webauthn.io/assertion

Body

```json
{
    "id": "AWU5bKbQIJGJfbRAJcuJKUVCKMOmIl7uDhHfIWiy0YBcn1fHz6zCWS91LFhDp4EUg-KntdmprFjfFSJmR9g",
    "rawId": "AWU5bKbQIJGJfbRAJcuJKUVCKMOmIl7uDhHfIWiy0YBcn1fHz6zCWS91LFhDp4EUg-KntdmprFjfFSJmR9g",
    "type": "public-key",
    "response": {
        "authenticatorData": "dKbqkhPJnC90siSSsyDPQCYqlMGpUKA5fyklC2CEHvAFXuktVg",
        "clientDataJSON": "eyJ0eXBlIjoid2ViYXV0aG4uZ2V0IiwiY2hhbGxlbmdlIjoiQXZtajZFQjZWSk5rZTM4UUptVzVNdzRiWURqTXhYeTBjaUR4TTg1NU5QbyIsIm9yaWdpbiI6Imh0dHBzOi8vd2ViYXV0aG4uaW8iLCJjcm9zc09yaWdpbiI6ZmFsc2UsImV4dHJhX2tleXNfbWF5X2JlX2FkZGVkX2hlcmUiOiJkbyBub3QgY29tcGFyZSBjbGllbnREYXRhSlNPTiBhZ2FpbnN0IGEgdGVtcGxhdGUuIFNlZSBodHRwczovL2dvby5nbC95YWJQZXgifQ",
        "signature": "MEUCIFwHpF16P1CiQ1wlstH3-hsoOl3fOXOAIa_9UiQHyMv5AiEAuAWOfdEmWfGzd8_5HTTIal3hft8mGnhnbKJJoG60Vwo",
        "userHandle": "tfMHAAAAAAAAAA"
    }
}
```



ClientDataJSON:

```json
{
    "type": "webauthn.get",
    "challenge": "Avmj6EB6VJNke38QJmW5Mw4bYDjMxXy0ciDxM855NPo",
    "origin": "https://webauthn.io",
    "crossOrigin": false,
    "extra_keys_may_be_added_here": "do not compare clientDataJSON against a template."
}
```

