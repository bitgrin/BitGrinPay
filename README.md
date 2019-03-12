# BitGrinPay

BitGrinPay is a specification for a simple way to send payments to users. This can be used to simplify payments from exchanges and pools where the recipient may not be able to easily accept HTTP(S) payouts.

## Overview

The BitGrin GUI wallet, Kingfish, implements BitGrinPay to simply orchestrate transaction file exchanges. Put simply, the system:

 - Provides an xbg:// link that embeds payer information including their name, logo, and a payout endpoint
 - The recipient uses this data to send an HTTPS API request to the payer which includes authentication credentials
 - The payer replies with a payment transaction send file
 - The recipient produces a response transaction file automatically, and sends the response back to the payer endpoint
 - The payer can then broadcast the response file, completing the transaction, and send a success/fail bit back to the recipient

 ## Motivation

 Currently, the simplest way to send and receive BitGrin payments is over HTTPS, which already orchestrates this process.
 However, HTTP/S transfers require the recipient to expose ports through their router, which may be difficult or even impossible for some users.

 Alternatively, a recipient could opt for receiving a transaction file and sending back a response, however this is a slow, manual process to perform over email or other means.

 Because the majority of exchanges and pools are perfectly capable of exposing HTTPS ports to expose their wallet listener, it is straight-forward for a user to make an HTTPS deposit. However, the withdraw process is extremely difficult for many users.

 Instead, BitGrinPay relies on the fact that the payer has the infrastructure required to expose API endpoints which orchestrate tx file exchange as HTTPS response that the recipient is easily capable of performing automatically.

 # Implementation

First, a payer produces a payout payload containing their payer information:

 ### payment_payload

  - `icon`: string - A user friendly 128x128 icon URL representing the payer
  - `name`: string - A user friendly name of the payer
  - `request_payout_url`: string - An HTTPS endpoint the user can POST to, in order to begin orchestrating slate exchange

 #### Example
 ```
 {
     icon: "https://example.com/icon.png",
     name: "My Example Exchange",
     request_payout_url: "https://example.com/api/request_payout",
 }
 ```

 This payload is then base64 encoded, and appended on to the end of an `xbg://` link on their website.

 #### Example
 ```
 xbg://IHsKICAgICBpY29uOiAiaHR0cHM6Ly9leGFtcGxlLmNvbS9pY29uLnBuZyIsCiAgICAgbmFtZTogIk15IEV4YW1wbGUgRXhjaGFuZ2UiLAogICAgIHJlcXVlc3RfcGF5b3V0X3VybDogImh0dHBzOi8vZXhhbXBsZS5jb20vYXBpL3JlcXVlc3RfcGF5b3V0IiwKIH0=
 ```

 [Request Payout](xbg://IHsKICAgICBpY29uOiAiaHR0cHM6Ly9leGFtcGxlLmNvbS9pY29uLnBuZyIsCiAgICAgbmFtZTogIk15IEV4YW1wbGUgRXhjaGFuZ2UiLAogICAgIHJlcXVlc3RfcGF5b3V0X3VybDogImh0dHBzOi8vZXhhbXBsZS5jb20vYXBpL3JlcXVlc3RfcGF5b3V0IiwKIH0=)

 When a user clicks this link, the Kingfish wallet and other supporting wallets should decode the payload, and present a confirmation prompt to proceed.

 Alternatively, the user can simply paste the base64 encoded string in to wallets that do not register the XBG scheme

---

### request_payout
This will be performed by the BitGrinPay client using the `request_payout_url` specified in the `payment_payload`

#### POST
Parameters:
 - `login`: string - A login username, email, etc
 - `secret`: string - A SHA256 hash of the user's passphrase
 
 Response:
  - `tx`: string - The content of a BitGrin send transaction file
  - `complete_payout_url`: The API endpoint of the payer the recipient should send their response back to
  - `login`: string - A login username, email, etc
  - `secret`: string - A SHA256 hash of the user's passphrase

---

### complete_payout
This will be performed by the BitGrinPay client using the `complete_payout_url` specified in the `request_payout` response

#### POST
Parameters:
 - `login`: string - A login username, email, etc
 - `secret`: string - A SHA256 hash of the user's passphrase
 - `response`: string - The content of a BitGrin response transaction file

Response:
 - `success`: bool - `true` after a successful broadcast. `false` after an unsuccessful broadcast.
 - `message`: string - A user friendly message to display to the user.

---

## Some considerations

Occassionaly, transactions will fail to reply with a valid response file. We recommend having an internal timeout managed server-side that will cancel the pending transaction after a short period of time. 30 seconds should be entirely adequate.

The same mechanics could be used for Grin with very minimal changes. Kingfish could add support for Grin and make the process easy for Grin users to receive payouts as well.
