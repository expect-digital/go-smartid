# Intro

![Smart-ID in Go language](https://github.com/dknight/go-smartid/files/smartid-gopher.png?raw=true)

Package smartid implements an imple interface in Go to work with the
Smart-ID API (https:www.smart-id.com). Smart-ID is used to easily and
safely authenticate and sign documents online using only a smart phone.
Smart-ID is a popular method in the Baltic countries of Estonia, Latvia,
and Lithuania for authenticating and signing documents online for banks,
social media, government offices, and other institutions.

## Installation

```sh
go get github.com/dknight/go-smartid
```

## Usage

The bare minimum required to make an authentication request. Demonstarates
synchronous way.

For more examples [see full docs](http://missing-yet).

```go
semid := NewSemanticIdentifier(IdentifierTypePNO, CountryEE, "30303039914")
client := NewClient("https:sid.demo.sk.ee/smart-id-rp/v2/", 5000)
request := AuthRequest{
	// Replace in production with real RelyingPartyUUID.
	RelyingPartyUUID: "00000000-0000-0000-0000-000000000000",
	// Replace in production with real RelyingPartyName.
	RelyingPartyName: "DEMO",
	// It is good to generate new has for security reasons.
	Hash: GenerateAuthHash(SHA512),
 	// We use personal ID as Identifier, also possible to use document number.
	Identifier: semid,
}

// This blocks thread until it completes
resp, err := client.AuthenticateSync(&request)
if err != nil {
	log.Fatalln(err)
}

if _, err := resp.Validate(); err != nil {
	log.Fatalln(err)
}

// It is also good to verify the certificate over secure. But it isn't
// mandatory, but strongly recommended.
// 
// If you always get an error: x509: certificate signed by unknown
// authority. Most probably you need install ca-certificates for
// example for GNU Linux.
// 
// sudo apt-get install ca-certificates
// sudo dnf install ca-certificates
certPaths := []string{"./certs/sid_demo_sk_ee_2022_PEM.crt"}
if ok, err := resp.Cert.Verify(certPaths); !ok {
 	log.Fatalln(err)
}

identity := resp.GetIdentity()
fmt.Println("Name:", identity.CommonName)
fmt.Println("Personal ID:", identity.SerialNumber)
fmt.Println("Country:", identity.Country)
// Output:
// Name: TESTNUMBER,QUALIFIED OK1
// Personal ID: PNOEE-30303039914
// Country: EE
```

### Async way using channel

Another example contains many more quest parameters for the signing method.
Sign and Authenticate methods are similar and you can use the same
AuthRequest parameters for both of them.

This examples is asynchronous uses channel.

```go
semid := NewSemanticIdentifier(IdentifierTypePNO, CountryEE, "30303039914")
client := NewClient("https://sid.demo.sk.ee/smart-id-rp/v2/", 5000)
request := AuthRequest{
	// Replace in production with real RelyingPartyUUID.
	RelyingPartyUUID: "00000000-0000-0000-0000-000000000000",
	// Replace in production with real RelyingPartyName.
	RelyingPartyName: "DEMO",
	// It is good to generate new has for security reasons.
	Hash: GenerateAuthHash(SHA384),
	// HashType should be the same as in GenerateAuthHash.
	HashType: SHA384,
	// We use personal ID as Identifier, also possible to use document
	// number.
	Identifier: semid,
	AuthType:   AuthTypeEtsi,
	CertificateLevel: CertLevelQualified,
	AllowedInteractionsOrder: []AllowedInteractionsOrder{
		{
			Type:          InteractionVerificationCodeChoice,
			DisplayText60: "Welcome to Smart-ID!",
		},
		{
			Type:          InteractionDisplayTextAndPIN,
			DisplayText60: "Welcome to Smart-ID!",
		},
	},
}

resp := <-client.Sign(&request)
if _, err := resp.Validate(); err != nil {
	log.Fatalln(err)
}

// It is also good to verify the certificate over secure. But it isn't
// mandatory, but strongly recommended.
//
// If you always get an error: x509: certificate signed by unknown
// authority. Most probably you need install ca-certificates for
// example for GNU Linux.
//
// sudo apt-get install ca-certificates
// sudo dnf install ca-certificates
certPaths := []string{"./certs/sid_demo_sk_ee_2022_PEM.crt"}
if ok, err := resp.Cert.Verify(certPaths); !ok {
	log.Fatalln(err)
}

identity := resp.GetIdentity()
fmt.Println("Name:", identity.CommonName)
fmt.Println("Personal ID:", identity.SerialNumber)
fmt.Println("Country:", identity.Country)
// Output:
// Name: TESTNUMBER,QUALIFIED OK1
// Personal ID: PNOEE-30303039914
// Country: EE
```

For more examples [see docs](http://missing-yet).

## What is not included

1. `:private/` endpoint.
2. Better certificated parsing and data extraction. You can get certificated
from response, verify and parse it in own way `response.Cert.GetX509Cert()`.
3. Smart-ID version V1.0 is not supported, only V2.0.

## Contribution

Any help is appreciated. Found a bug, typo, inaccuracy, etc. Please do not 
hesitate and make pull request or issue.

## License 

MIT (2022)
