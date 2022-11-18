## cas_authenticate
* get ticket from url parameter and get cookie MOD_AUTH_CAS_S
* if ticket is not null
  * isValidCASTicket -> getResponseFromServer 
    1. send POST (SAML soap) or GET (url parameter) request by cas validateURL to verify ticket
    2. parse response for validateURL
  * Ticket is valid
    1. create MOD_AUTH_CAS_S cookie
    2. response 302 with redirect location to original URL
  * Ticket is invalid
    * If cookie MOD_AUTH_CAS_S is null, respond 403
    * else, handle cookie MOD_AUTH_CAS_S
* if cookie MOD_AUTH_CAS_S is not null (ticket is null or invalid)
  * isValidCASCookie check cookies is valid or not
  * if invalid, respond 302 with redirect location to cas login URL
  * if valid, respond the response of original request URL
* if cookie MOD_AUTH_CAS_S is null  (ticket is null or invalid)
  * respond 302 with redirect location to cas login URL
