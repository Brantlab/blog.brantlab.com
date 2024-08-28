---
title: "VMware Explore Hackathon 24 - Automating a phone IVR Message"
author: "Justin Brant"
cover: "/img/Hackathon24/Workflow.png"
tags: ["Automation", N8N, ChatGPT]
date: 2024-08-26
draft: false
---


## What is the goal?

This year for the hackathon at VMware Explore I was unsure I would be able to attend so I ended up being a walk in and sometimes those are the best projects and networking oppurtunites. My team originally was just talking about life with everything from Firefighting to the spiders in austrailia. PowerCLI man came over and was like hey why don't you guys do something with ChatGPT. We decided to use a project I had actually worked on prior to arriving at the hackathon. 

## Whats the problem?

My lovely wife works for a grain equity who is still modernizing their entire business. Let me set the stage for you, You walk into their office and they will have wood paneling, They had a phone system but was limited to each facility and was not interc-connected between sites. Not related but made this request possible is we migrated them off old copper and onto a VOIP solution through RingCentral. Before this they would record a voicemail message every night with grain bid information for farmers. Essentially its their version of the stock market. This was a very manual process. 

## How do we solve this? 

On their website for the company they pay for a service called DTN that allows us API calls to grab that grain data and show it on their website. Excellent. This helps us by giving us our first step and didn't hard stop this idea. My idea was to grab data, format it, and upload it to RingCentrals IVR text to speech API for IVR Prompts. We have the plan. 

## Lets discuss the issue with said plan.

I don't want to get out into the weeds yet on the process that actually works but this is pretty relevent. I had all this setup and running for a few weeks but RingCentral's IVR system leaves a little to be desired along with their documentation. We would push the same prompt via API and it would be reflected in the UI and such but it would never update when you called the system. There was no errors. This could be some random charcters in the message it did not like. We sturggled with commas for awhile then we swapped to periods to create "breaths" in the correct places for the message but then that all but stopped working. We decided that we needed to move to a different automation. 

## Integration hell? New plan? 

Ok so now begins the rabbit hole of dealing with binary WAV files and Azure TTS services and learning all the things. 

{{< figure src="/img/Hackathon24/Workflow.png" width=75% layout="responsive" >}}


The new goal was to use RingCentral prompt with a crafted wav file and upload it to RingCentral and design some sort of approval process. The wife's company does have a coporate office 365 and are utilizing teams for that. I thought this would be a perfect and elegant solution to make a room so select people could approve via card in teams if she was off, sick, etc. 

Ok I seem to have a plan. Lets go ahead and work through this. 

## Phase 1

We began by creating a new N8N workflow with a time based tirgger. This only needs kicked off once a day at 2:45 PM EST time. This leads right into a API call to DTN that gets grain data. This data come back as JSON and I will include a snippet of that data. 

{{< figure src="/img/Hackathon24/Phase1.png" width=35% layout="responsive" >}}

{{< details title="JSON of Data example" >}}
``` json
[
  {
    "allowTransactions": true,
    "basisPrice": -0.4,
    "cashPrice": 3.22,
    "commodityDisplayName": "Corn",
    "commodityId": 4,
    "contractDeliveryLabel": "Cash",
    "contractMonthCode": "20240831",
    "conversionUsed": "0",
    "convertedPrice": null,
    "currency": "USD",
    "deliveryPeriod": {
      "end": "2024-08-31T05:00:00Z",
      "start": "2024-08-01T05:00:00Z"
    },
    "futuresChange": "0'4",
    "futuresQuote": "362'4",
    "id": 4101519,
    "location": {
      "id": 8442,
      "name": "Scott / Van Wert"
    },
    "primaryPrice": {
      "basisPrice": -0.4,
      "cashPrice": 3.22,
      "currency": "USD",
      "unitOfMeasure": "Bushels"
    },
    "realTime": false,
    "settlePrice": "362'0",
    "symbol": "@C4U",
    "unitOfMeasure": "Bushels"
  },
  {
    "allowTransactions": true,
    "basisPrice": -0.45,
    "cashPrice": 3.42,
    "commodityDisplayName": "Corn",
    "commodityId": 4,
    "contractDeliveryLabel": "New 24",
    "contractMonthCode": "20241031",
    "conversionUsed": "0",
    "convertedPrice": null,
    "currency": "USD",
    "deliveryPeriod": {
      "end": "2024-10-31T05:00:00Z",
      "start": "2024-10-01T05:00:00Z"
    },
    "futuresChange": "0'2",
    "futuresQuote": "386'6",
    "id": 4128397,
    "location": {
      "id": 8442,
      "name": "Scott / Van Wert"
    },
    "primaryPrice": {
      "basisPrice": -0.45,
      "cashPrice": 3.42,
      "currency": "USD",
      "unitOfMeasure": "Bushels"
    },
    "realTime": false,
    "settlePrice": "386'4",
    "symbol": "@C4Z",
    "unitOfMeasure": "Bushels"
  },
  {
    "allowTransactions": true,
    "basisPrice": -0.4,
    "cashPrice": 3.66,
    "commodityDisplayName": "Corn",
    "commodityId": 4,
    "contractDeliveryLabel": "Jan '25",
    "contractMonthCode": "20250131",
    "conversionUsed": "0",
    "convertedPrice": null,
    "currency": "USD",
    "deliveryPeriod": {
      "end": "2025-01-31T06:00:00Z",
      "start": "2025-01-01T06:00:00Z"
    },
    "futuresChange": "0'4",
    "futuresQuote": "405'6",
    "id": 4166157,
    "location": {
      "id": 8442,
      "name": "Scott / Van Wert"
    },
    "primaryPrice": {
      "basisPrice": -0.4,
      "cashPrice": 3.66,
      "currency": "USD",
      "unitOfMeasure": "Bushels"
    },
    "realTime": false,
    "settlePrice": "405'2",
    "symbol": "@C5H",
    "unitOfMeasure": "Bushels"
  },
  {
    "allowTransactions": true,
    "basisPrice": 0,
    "cashPrice": 9.8,
    "commodityDisplayName": "Soybeans",
    "commodityId": 30,
    "contractDeliveryLabel": "Cash",
    "contractMonthCode": "20240831",
    "conversionUsed": "0",
    "convertedPrice": null,
    "currency": "USD",
    "deliveryPeriod": {
      "end": "2024-08-31T05:00:00Z",
      "start": "2024-08-01T05:00:00Z"
    },
    "futuresChange": "-0'2",
    "futuresQuote": "980'4",
    "id": 4332967,
    "location": {
      "id": 8442,
      "name": "Scott / Van Wert"
    },
    "primaryPrice": {
      "basisPrice": 0,
      "cashPrice": 9.8,
      "currency": "USD",
      "unitOfMeasure": "Bushels"
    },
    "realTime": false,
    "settlePrice": "980'6",
    "symbol": "@S4X",
    "unitOfMeasure": "Bushels"
  },
  {
    "allowTransactions": true,
    "basisPrice": -0.1,
    "cashPrice": 9.7,
    "commodityDisplayName": "Soybeans",
    "commodityId": 30,
    "contractDeliveryLabel": "FH Sept",
    "contractMonthCode": "20240930",
    "conversionUsed": "0",
    "convertedPrice": null,
    "currency": "USD",
    "deliveryPeriod": {
      "end": "2024-09-30T05:00:00Z",
      "start": "2024-09-01T05:00:00Z"
    },
    "futuresChange": "-0'2",
    "futuresQuote": "980'4",
    "id": 4332957,
    "location": {
      "id": 8442,
      "name": "Scott / Van Wert"
    },
    "primaryPrice": {
      "basisPrice": -0.1,
      "cashPrice": 9.7,
      "currency": "USD",
      "unitOfMeasure": "Bushels"
    },
    "realTime": false,
    "settlePrice": "980'6",
    "symbol": "@S4X",
    "unitOfMeasure": "Bushels"
  },
  {
    "allowTransactions": true,
    "basisPrice": -0.6,
    "cashPrice": 9.2,
    "commodityDisplayName": "Soybeans",
    "commodityId": 30,
    "contractDeliveryLabel": "New 24",
    "contractMonthCode": "20241130",
    "conversionUsed": "0",
    "convertedPrice": null,
    "currency": "USD",
    "deliveryPeriod": {
      "end": "2024-11-30T06:00:00Z",
      "start": "2024-11-01T05:00:00Z"
    },
    "futuresChange": "-0'2",
    "futuresQuote": "980'4",
    "id": 4332952,
    "location": {
      "id": 8442,
      "name": "Scott / Van Wert"
    },
    "primaryPrice": {
      "basisPrice": -0.6,
      "cashPrice": 9.2,
      "currency": "USD",
      "unitOfMeasure": "Bushels"
    },
    "realTime": false,
    "settlePrice": "980'6",
    "symbol": "@S4X",
    "unitOfMeasure": "Bushels"
  },
  {
    "allowTransactions": true,
    "basisPrice": -0.45,
    "cashPrice": 9.53,
    "commodityDisplayName": "Soybeans",
    "commodityId": 30,
    "contractDeliveryLabel": "Jan '25",
    "contractMonthCode": "20250131",
    "conversionUsed": "0",
    "convertedPrice": null,
    "currency": "USD",
    "deliveryPeriod": {
      "end": "2025-01-31T06:00:00Z",
      "start": "2025-01-01T06:00:00Z"
    },
    "futuresChange": "-0'4",
    "futuresQuote": "997'6",
    "id": 4166162,
    "location": {
      "id": 8442,
      "name": "Scott / Van Wert"
    },
    "primaryPrice": {
      "basisPrice": -0.45,
      "cashPrice": 9.53,
      "currency": "USD",
      "unitOfMeasure": "Bushels"
    },
    "realTime": false,
    "settlePrice": "998'2",
    "symbol": "@S5F",
    "unitOfMeasure": "Bushels"
  },
  {
    "allowTransactions": true,
    "basisPrice": -0.5,
    "cashPrice": 4.46,
    "commodityDisplayName": "Wheat",
    "commodityId": 40,
    "contractDeliveryLabel": "Cash",
    "contractMonthCode": "20240831",
    "conversionUsed": "0",
    "convertedPrice": null,
    "currency": "USD",
    "deliveryPeriod": {
      "end": "2024-08-31T05:00:00Z",
      "start": "2024-08-01T05:00:00Z"
    },
    "futuresChange": "-1'6",
    "futuresQuote": "496'2",
    "id": 4101522,
    "location": {
      "id": 8442,
      "name": "Scott / Van Wert"
    },
    "primaryPrice": {
      "basisPrice": -0.5,
      "cashPrice": 4.46,
      "currency": "USD",
      "unitOfMeasure": "Bushels"
    },
    "realTime": false,
    "settlePrice": "498'0",
    "symbol": "@W4U",
    "unitOfMeasure": "Bushels"
  },
  {
    "allowTransactions": true,
    "basisPrice": -0.5,
    "cashPrice": 4.95,
    "commodityDisplayName": "Wheat",
    "commodityId": 40,
    "contractDeliveryLabel": "Jan '25",
    "contractMonthCode": "20250131",
    "conversionUsed": "0",
    "convertedPrice": null,
    "currency": "USD",
    "deliveryPeriod": {
      "end": "2025-01-31T06:00:00Z",
      "start": "2025-01-01T06:00:00Z"
    },
    "futuresChange": "-1'2",
    "futuresQuote": "545'0",
    "id": 4166165,
    "location": {
      "id": 8442,
      "name": "Scott / Van Wert"
    },
    "primaryPrice": {
      "basisPrice": -0.5,
      "cashPrice": 4.95,
      "currency": "USD",
      "unitOfMeasure": "Bushels"
    },
    "realTime": false,
    "settlePrice": "546'2",
    "symbol": "@W5H",
    "unitOfMeasure": "Bushels"
  },
  {
    "allowTransactions": true,
    "basisPrice": -0.55,
    "cashPrice": 5.07,
    "commodityDisplayName": "Wheat",
    "commodityId": 40,
    "contractDeliveryLabel": "July '25",
    "contractMonthCode": "20250731",
    "conversionUsed": "0",
    "convertedPrice": null,
    "currency": "USD",
    "deliveryPeriod": {
      "end": "2025-07-31T05:00:00Z",
      "start": "2025-07-01T05:00:00Z"
    },
    "futuresChange": "-1'4",
    "futuresQuote": "562'2",
    "id": 4245212,
    "location": {
      "id": 8442,
      "name": "Scott / Van Wert"
    },
    "primaryPrice": {
      "basisPrice": -0.55,
      "cashPrice": 5.07,
      "currency": "USD",
      "unitOfMeasure": "Bushels"
    },
    "realTime": false,
    "settlePrice": "563'6",
    "symbol": "@W5N",
    "unitOfMeasure": "Bushels"
  },
  {
    "allowTransactions": true,
    "basisPrice": 0.02,
    "cashPrice": 9.83,
    "commodityDisplayName": "Soybeans",
    "commodityId": null,
    "contractDeliveryLabel": "Cash",
    "contractMonthCode": "20240831",
    "conversionUsed": "0",
    "convertedPrice": null,
    "currency": "USD",
    "deliveryPeriod": {
      "end": "2024-08-31T05:00:00Z",
      "start": "2024-08-01T05:00:00Z"
    },
    "futuresChange": "-0'2",
    "futuresQuote": "980'6",
    "id": 4101524,
    "location": {
      "id": 8443,
      "name": "Delphos"
    },
    "primaryPrice": {
      "basisPrice": 0.02,
      "cashPrice": 9.83,
      "currency": "USD",
      "unitOfMeasure": "Bushels"
    },
    "realTime": false,
    "settlePrice": "980'6",
    "symbol": "@S4X",
    "unitOfMeasure": "Bushels"
  },
  {
    "allowTransactions": true,
    "basisPrice": -0.13,
    "cashPrice": 9.68,
    "commodityDisplayName": "Soybeans",
    "commodityId": null,
    "contractDeliveryLabel": "FH Sept",
    "contractMonthCode": "20240930",
    "conversionUsed": "0",
    "convertedPrice": null,
    "currency": "USD",
    "deliveryPeriod": {
      "end": "2024-09-30T05:00:00Z",
      "start": "2024-09-01T05:00:00Z"
    },
    "futuresChange": "-0'2",
    "futuresQuote": "980'6",
    "id": 4110668,
    "location": {
      "id": 8443,
      "name": "Delphos"
    },
    "primaryPrice": {
      "basisPrice": -0.13,
      "cashPrice": 9.68,
      "currency": "USD",
      "unitOfMeasure": "Bushels"
    },
    "realTime": false,
    "settlePrice": "980'6",
    "symbol": "@S4X",
    "unitOfMeasure": "Bushels"
  },
  {
    "allowTransactions": true,
    "basisPrice": -0.38,
    "cashPrice": 9.43,
    "commodityDisplayName": "Soybeans",
    "commodityId": null,
    "contractDeliveryLabel": "New 24",
    "contractMonthCode": "20241031",
    "conversionUsed": "0",
    "convertedPrice": null,
    "currency": "USD",
    "deliveryPeriod": {
      "end": "2024-10-31T05:00:00Z",
      "start": "2024-10-01T05:00:00Z"
    },
    "futuresChange": "-0'2",
    "futuresQuote": "980'6",
    "id": 4146101,
    "location": {
      "id": 8443,
      "name": "Delphos"
    },
    "primaryPrice": {
      "basisPrice": -0.38,
      "cashPrice": 9.43,
      "currency": "USD",
      "unitOfMeasure": "Bushels"
    },
    "realTime": false,
    "settlePrice": "980'6",
    "symbol": "@S4X",
    "unitOfMeasure": "Bushels"
  },
  {
    "allowTransactions": true,
    "basisPrice": -0.18,
    "cashPrice": 9.8,
    "commodityDisplayName": "Soybeans",
    "commodityId": null,
    "contractDeliveryLabel": "Jan 25",
    "contractMonthCode": "20250131",
    "conversionUsed": "0",
    "convertedPrice": null,
    "currency": "USD",
    "deliveryPeriod": {
      "end": "2025-01-31T06:00:00Z",
      "start": "2025-01-01T06:00:00Z"
    },
    "futuresChange": "-0'4",
    "futuresQuote": "998'2",
    "id": 4166167,
    "location": {
      "id": 8443,
      "name": "Delphos"
    },
    "primaryPrice": {
      "basisPrice": -0.18,
      "cashPrice": 9.8,
      "currency": "USD",
      "unitOfMeasure": "Bushels"
    },
    "realTime": false,
    "settlePrice": "998'2",
    "symbol": "@S5F",
    "unitOfMeasure": "Bushels"
  },
  {
    "allowTransactions": true,
    "basisPrice": -0.13,
    "cashPrice": 9.68,
    "commodityDisplayName": "Soybeans",
    "commodityId": null,
    "contractDeliveryLabel": "Cash",
    "contractMonthCode": "20240831",
    "conversionUsed": "0",
    "convertedPrice": null,
    "currency": "USD",
    "deliveryPeriod": {
      "end": "2024-08-31T05:00:00Z",
      "start": "2024-08-01T05:00:00Z"
    },
    "futuresChange": "-0'2",
    "futuresQuote": "980'6",
    "id": 4101525,
    "location": {
      "id": 8444,
      "name": "Decatur"
    },
    "primaryPrice": {
      "basisPrice": -0.13,
      "cashPrice": 9.68,
      "currency": "USD",
      "unitOfMeasure": "Bushels"
    },
    "realTime": false,
    "settlePrice": "980'6",
    "symbol": "@S4X",
    "unitOfMeasure": "Bushels"
  },
  {
    "allowTransactions": true,
    "basisPrice": -0.18,
    "cashPrice": 9.63,
    "commodityDisplayName": "Soybeans",
    "commodityId": null,
    "contractDeliveryLabel": "FH Sept",
    "contractMonthCode": "20240930",
    "conversionUsed": "0",
    "convertedPrice": null,
    "currency": "USD",
    "deliveryPeriod": {
      "end": "2024-09-30T05:00:00Z",
      "start": "2024-09-01T05:00:00Z"
    },
    "futuresChange": "-0'2",
    "futuresQuote": "980'6",
    "id": 4110669,
    "location": {
      "id": 8444,
      "name": "Decatur"
    },
    "primaryPrice": {
      "basisPrice": -0.18,
      "cashPrice": 9.63,
      "currency": "USD",
      "unitOfMeasure": "Bushels"
    },
    "realTime": false,
    "settlePrice": "980'6",
    "symbol": "@S4X",
    "unitOfMeasure": "Bushels"
  },
  {
    "allowTransactions": true,
    "basisPrice": -0.36,
    "cashPrice": 9.45,
    "commodityDisplayName": "Soybeans",
    "commodityId": null,
    "contractDeliveryLabel": "New 24",
    "contractMonthCode": "20241031",
    "conversionUsed": "0",
    "convertedPrice": null,
    "currency": "USD",
    "deliveryPeriod": {
      "end": "2024-10-31T05:00:00Z",
      "start": "2024-10-01T05:00:00Z"
    },
    "futuresChange": "-0'2",
    "futuresQuote": "980'6",
    "id": 4146102,
    "location": {
      "id": 8444,
      "name": "Decatur"
    },
    "primaryPrice": {
      "basisPrice": -0.36,
      "cashPrice": 9.45,
      "currency": "USD",
      "unitOfMeasure": "Bushels"
    },
    "realTime": false,
    "settlePrice": "980'6",
    "symbol": "@S4X",
    "unitOfMeasure": "Bushels"
  },
  {
    "allowTransactions": true,
    "basisPrice": -0.13,
    "cashPrice": 9.85,
    "commodityDisplayName": "Soybeans",
    "commodityId": null,
    "contractDeliveryLabel": "Jan 25",
    "contractMonthCode": "20250131",
    "conversionUsed": "0",
    "convertedPrice": null,
    "currency": "USD",
    "deliveryPeriod": {
      "end": "2025-01-31T06:00:00Z",
      "start": "2025-01-01T06:00:00Z"
    },
    "futuresChange": "-0'4",
    "futuresQuote": "998'2",
    "id": 4166173,
    "location": {
      "id": 8444,
      "name": "Decatur"
    },
    "primaryPrice": {
      "basisPrice": -0.13,
      "cashPrice": 9.85,
      "currency": "USD",
      "unitOfMeasure": "Bushels"
    },
    "realTime": false,
    "settlePrice": "998'2",
    "symbol": "@S5F",
    "unitOfMeasure": "Bushels"
  }
]
```
{{< /details >}}


Data is in a typical structure and we will need to filter the data down to what we need. We do this with a bit of javascript in the next step. 


## Phase 2

{{< figure src="/img/Hackathon24/Phase2.png" width=65% layout="responsive" >}}

We then go ahead and format the data coming off. This was one fo the first interactions with ChatGPT on this project. I fed it the phone prompt my user had and gave it some expectations on where numbers needed to land and with some of the data from the JSON above. It was then able to write this javascript. To further help me after I start writing this I had it write its own blurb about its code. Note I did verify the code and verified nothing malicious was pertained in it. 

> "In this n8n workflow, I created a function to dynamically generate SSML (Speech Synthesis Markup Language) for a TTS (Text-to-Speech) system that reports daily commodity prices. The workflow processes data by extracting and formatting relevant information, such as cash prices and futures changes, and composes it into a natural-sounding spoken report. The code includes logic to handle various cases, like specific locations or contract delivery labels, ensuring that the information is accurately conveyed in a user-friendly format. This automation is tailored for an IVR (Interactive Voice Response) system, enabling users to receive up-to-date commodity prices through voice prompts." 
>
> – ChatGPT 4.0

{{< details title="Javascript for formatting IVR Prompt for Azure TTS" >}}
``` javascript
function getFuturesDirection(futuresChange) {
    const futuresNumber = futuresChange.split("'")[0];
    const isNegative = futuresChange.startsWith("-");
    
    let direction = "up";
    if (futuresNumber === "0" || isNegative) {
        direction = "down";
    }
    
    return `${direction} ${futuresNumber.replace('-', '')}.`;
}

function getTruncatedInteger(decimalValue) {
    return Math.floor(decimalValue);
}

const date = new Date();
const month = date.toLocaleString('default', { month: 'long' });
const day = date.getDate();
const year = date.getFullYear();

let ssml = `<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
  <voice name="en-US-JennyNeural">
    Closes for ${month} ${day}, ${year}. `;

items.forEach(item => {
    const commodity = item.json;
    const cashDollar = getTruncatedInteger(commodity.cashPrice);
    const cashCents = Math.round((commodity.cashPrice - cashDollar) * 100);
    const cashFuture = getFuturesDirection(commodity.futuresChange);
    const displayName = commodity.commodityDisplayName;
    const location = commodity.location.name.includes("Scott / Van Wert") ? "" : `to ${commodity.location.name}`;
    
    let label;
    if (commodity.contractDeliveryLabel === "Cash") {
        label = "Cash";
    } else {
        const dateCode = commodity.contractMonthCode.toString();
        const yearPart = dateCode.slice(0, 4);
        const monthPart = dateCode.slice(4, 6);
        const dateObject = new Date(`${yearPart}-${monthPart}-01`);
        label = `${dateObject.toLocaleString('default', { month: 'long' })},`; // Adding comma after the month
    }

    // Limit Delphos and Decatur to Cash only
    if ((commodity.location.name.includes("Delphos") || commodity.location.name.includes("Decatur")) && label !== "Cash") {
        return;
    }
    
    ssml += `${displayName} ${location} ${label} ${cashDollar} dollars and ${cashCents} cents ${cashFuture} `;
});

ssml += `Commodity sales are based off the Redacted Website at Redacted.com 
  Press star to return to the main menu or press pound to hear this information again
  </voice></speak>`;

```
{{< /details >}}

{{< figure src="/img/Hackathon24/functionATTS.png" width=75% layout="responsive" >}}


Once this has happened I went ahead and used ChatGPT to help me get the info over to Azure TTS which was pretty uneventful since that is a simple restAPI post call to https://eastus.tts.speech.microsoft.com/cognitiveservices/v1.

{{< collapsible-image 
    title="I have provided a screenshot with headers that I used to get my WAV file. Click arrow to expand." 
    src="/img/Hackathon24/AzureTTS.png" 
    width="75%" 
    layout="responsive" 
    alt="Azure TTS Screenshot"
>}}

While this is all happening we go ahead and work in phase 3 for our approval stages for our message. 

## Phase 3

{{< figure src="/img/Hackathon24/Phase3.png" width=75% layout="responsive" >}}

Phase 3 was unique as we decided to add in human intervention at least during this time so we can verify it has the correct information for at least during our proof of concept period. We may move away from the physical approval method at some point and just post the info into the teams space. 

This was also new for me as I had never worked with a return webhook or teams card to make it look nice for the end user. 

>	“This script is a great example of leveraging JavaScript within n8n to create a seamless integration between IVR systems and Microsoft Teams. It dynamically constructs speech synthesis for commodity prices and sends an approval workflow to Teams, allowing for real-time decision-making. The code elegantly handles various edge cases, ensuring accurate and contextually appropriate communication across platforms.”
>
>	– ChatGPT 4.0


First step in the phase uses some javascript again from ChatGPT. 

{{< details title="Javascript for formatting for Teams Approval" >}}
``` javascript
// Function to get futures direction
function getFuturesDirection(futuresChange) {
    const futuresNumber = futuresChange.split("'")[0];
    const isNegative = futuresChange.startsWith("-");
    
    let direction = "up";
    if (futuresNumber === "0" || isNegative) {
        direction = "down";
    }
    
    return `${direction} ${futuresNumber.replace('-', '')}`;
}

// Function to truncate the integer
function getTruncatedInteger(decimalValue) {
    return Math.floor(decimalValue);
}

const date = new Date();
const month = date.toLocaleString('default', { month: 'long' });
const day = date.getDate();
const year = date.getFullYear();

let phonePhrase = `Closes. for. ${month}. ${day}.. ${year}...\n`;

items.forEach(item => {
    const commodity = item.json;
    const cashDollar = getTruncatedInteger(commodity.cashPrice);
    const cashCents = Math.round((commodity.cashPrice - cashDollar) * 100);
    const cashFuture = getFuturesDirection(commodity.futuresChange);
    const displayName = commodity.commodityDisplayName;
    const location = commodity.location.name.includes("Scott / Van Wert") ? "" : `to, ${commodity.location.name}`;
    
    let label;
    if (commodity.contractDeliveryLabel === "Cash") {
        label = "Cash";
    } else {
        const dateCode = commodity.contractMonthCode.toString();
        const dateString = `${dateCode.slice(0, 4)}-${dateCode.slice(4, 6)}-${dateCode.slice(6, 8)}`;
        const dateObject = new Date(dateString);
        label = dateObject.toLocaleString('default', { month: 'long' });
    }

    // Limit Delphos and Decatur to Cash only
    if ((commodity.location.name.includes("Delphos") || commodity.location.name.includes("Decatur")) && label !== "Cash") {
        return;
    }
    
    const phrase = `${displayName} ${location}. ${label}. ${cashDollar} dollars. and. ${cashCents} cents. ${cashFuture}..\n`;
    phonePhrase += phrase;
});

phonePhrase += "Commodity sales are based off Redacted Website at Redacted dot com...\nPress star to return to the main menu. Or press pound to hear this information again";

// Get the resume URLs for approval and rejection
const resumeUrlApprove = $execution.resumeUrl + "?response=approve";
const resumeUrlReject = $execution.resumeUrl + "?response=reject";

// Construct the JSON payload for Microsoft Teams with Markdown
const teamsMessage = {
  "type": "message",
  "attachments": [
    {
      "contentType": "application/vnd.microsoft.card.adaptive",
      "content": {
        "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
        "type": "AdaptiveCard",
        "version": "1.2",
        "body": [
          {
            "type": "TextBlock",
            "text": "Please approve the following data update:",
            "wrap": true
          },
          {
            "type": "TextBlock",
            "text": phonePhrase.replace(/\n/g, "\n\n"),
            "wrap": true
          }
        ],
        "actions": [
          {
            "type": "Action.OpenUrl",
            "title": "Approve",
            "url": resumeUrlApprove
          },
          {
            "type": "Action.OpenUrl",
            "title": "Reject",
            "url": resumeUrlReject
          }
        ]
      }
    }
  ]
};

return [{ json: teamsMessage }];
```
{{< /details >}}

Once this format is complete it posts that into the chat system. 

## insert image from my lovely wife once she sends it to me. 

The N8N workflow then waits for a webhook back. That will come back with a JSON reply with either approve and deny message. We then go ahead and hit that switch and determines which way we are going to go depending on the selection. 

Once that has been determined we send a message back to teams verifying that it was good to go and moved on to connecting to RingCentral to use JWT (JSON Web Token) to get a token for us to be able to communicate with Ring Central. ChatGPT was lovely at providing me the info needed on how JWT tokens work.

> "JSON Web Tokens (JWT) are a secure and compact way to transmit information between parties as a JSON object. They consist of three parts: a header, a payload, and a signature, which ensure data integrity and authenticity. JWTs are commonly used for authorization in web applications, allowing servers to verify user identity without storing session data. They are versatile, being easy to include in URLs, headers, or cookies, and support both signing and encryption for added security. Overall, JWTs provide a stateless and efficient solution for handling user authentication and authorization."
>
>	– ChatGPT 4.0

Once that token comes in we go ahead and stirp out the data we don't need and carry that onto where we merge all of our data together to simplify the request we are going to craft to send off to RingCentral. 

## Phase 4

It seems like it has taken forever to get here but in this stage we cover uploading the wav data we got from Azure TTS. Once that has been uploaded RingCentral provides a URI and ID number that we translate into the last call that offically pushes that voice file into production. That last call goes into the IVR menu itself and sets it to the new prompt URI. You can then call their main number and click option 3 and it will play that lovely WAV file for you. 

{{< figure src="/img/Hackathon24/Phase4.png" width=75% layout="responsive" >}}

{{< details title="request to upload WAV IVR menu file" >}}
``` JSON
    {
      "parameters": {
        "method": "POST",
        "url": "https://platform.RingCentral.com/restapi/v1.0/account/~/ivr-prompts",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "Authorization",
              "value": "=Bearer  {{ $json.access_token }}"
            },
            {
              "name": "Content-Type",
              "value": "multipart/form-data"
            }
          ]
        },
        "sendBody": true,
        "contentType": "multipart-form-data",
        "bodyParameters": {
          "parameters": [
            {
              "name": "name",
              "value": "=Prompt"
            },
            {
              "parameterType": "formBinaryData",
              "name": "attachment",
              "inputDataFieldName": "=data"
            }
          ]
        },
        "options": {}
      },
```
{{< /details >}}

{{< details title="request to update IVR menu URI" >}}
``` json

     "id": "1c4b87d1-0f20-405a-94fd-5f769d25254c",
      "name": "RC Upload files",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.2,
      "position": [
        2640,
        -200
      ]
    },
    {
      "parameters": {
        "method": "PUT",
        "url": "https://platform.RingCentral.com/restapi/v1.0/account/~/ivr-menus/<ID>",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "Content-Type",
              "value": "application/json"
            },
            {
              "name": "Authorization",
              "value": "=Bearer {{ $('Merge').item.json.access_token }}"
            }
          ]
        },
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={\n  \"prompt\": {\n    \"audio\": {\n      \"uri\": \"{{ $json.uri }}\",\n      \"id\": \"{{ $json.id}}\"\n    },\n    \"mode\": \"Audio\"\n  },\n  \"id\": \"<ID>\"\n}",
        "options": {}
```
{{< /details >}}


##  All done? Not quite.....

One challenge that neither ChatGPT or I could solve was the WAV file is always named "v1" and if we changed the name with N8N we lost the binary data. The prompt upload would no respect the name field and alwaus named the prompt the same as the file. We then had multiple v1 files sitting in the UI in RingCentral and that just isn't condusive if there was an issue or if they had other prompts they wanted to keep around. 

## Phase 5 

{{< figure src="/img/Hackathon24/Phase5.png" width=75% layout="responsive" >}}

This was a bit of a challenge due to having to array data and bringing it into flat data so we had to break it down and then do another query to determine what was actually in used. 

The flatten looks similar in both functions

``` javascript
const ivrMenus = $json.records;
return ivrMenus.map(menu => {
  return { json: menu };
});
```

Once that data is flat we can then filter and compare data. We apply a simple filter that filters out any TTS prompts so we are left with only audio prompts. This allows us to compare the IVR prompt audio file URIs and determine which ones are no longer used and remove them to keep the environment clean. 


## Synopsis
This was a great project even if there were times I thought it was taking years off my life but with the help of ChatGPT as almost dare I say a colleague helping step through, talk, and learn. I learned about JWT's, formatting data, Javascript, and troubleshooting API challenges with limited support. 

Check them out here https://n8n.io/