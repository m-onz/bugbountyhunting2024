
# bug bounty hunting with LLMs

# about me

I work in the analytics and tagging team within NLE / IA.
I like web application security and AI.
I'm a software developer, engineer and sometimes a mad scientist. 
I've always been interested in infosec but never employed as my main job.
I like programming, distributed systems, linux, BSD & much more.

# In my spare time:

I'm part of the algorave scene as an audio visual artist and VJ
I run an algorithmic music event and make visuals using open source software: puredata.info
I'm performing tonight at Corsica Studio's in London at an algorave
I also make music the old fashion way and play many instruments including the drums and sax.

# Bug bounty hunting

I am a very unsuccessful bug bounty hunter using it to learn and understand web application security.
I will tell all the ways I've been unsuccessful and hopefully it will help you avoid my mistakes!

# bug bounty hunting and VDP

I think it started as safe haven for ethical hackers to report security findings but has now turned into a vibrant eco-system.

There are bug bounty platforms that act as the mediators between companies and hackers.

The scope is clearly defined and genuine findings can earn big cash.

Sounds appealing? 

Certainly to me but its highly competitive.

There are public and private programs

# types of programs

private
public

private programs are exclusive and only selected people get to do them.

public programs are open to pretty much anybody.

# differences between pentesting and bug bounty hunting

# pentesting

-> low hanging fruit exists
-> scanners can be useful
-> can access developers to ask questions
-> can sometimes audit source code
-> no other attackers are usually competing with you
-> the application has not been audited so more bugs and issues likely exist

# bug bounty hunting

-> no low hanging fruit
-> scanners don't reveal much
-> hardened targets that may have gone through private programs
-> huge competition globally and from seasoned full time experts

# bug bounty strategies that don't work

Here are some strategies that deffinitely don't work for bug bounty hunting...
picking targets at random.
scanning.
not being organised.
getting bored and picking other targets. / giving up too easily.
not having any criteria for applications to target.
not taking the time to understand how an application works.


# the strategy 2.0 I'm working on

I have a critera for applications:
    newness
    complexity
    size

# bug bounty approaches

pentesting approach -> pick a target and use common tools: burp suite / etc / http proxy / browser forwarding proxy

    heavy scanning of a target is possible on a willing target & with consent
    bug bounty targets use web application firewalls, bot defence, rate limiting.
    if a target blocks your IP or uses other defences the scan will be ineffective




cli driven lateral first scanning -> tomnomnom -> etc

AI powered approach <--- the subject of this talk

# trands in bug bounty hunting

asset discovery, nuclei
automation, use of AI

# LLMS

chat bots.. predicting the next word on steroids

emergent capabilities ->

what are LLMs -> garbage in / garbage out

there strengths and weakness'

hallucination or bullshit?

# --->

take a list of target domains from <github link>
optionally expand out more targets using subdomain enumeration

# target discovery : need in a haystack

1 .chunk the list and ask an LLM which targets fit a certain criteria [ ]

--> the LLM provider may include the ability to search the internet which will increase the usefulness of this approach.

2 .fetch each URL... download the content into folders. Then ask an LLM about chunks of the downloaded content

--> show some code examples of this approach.

# finding bugs in applications

Once you have discovered a target that fits your criteria...

Using a browser forwarding proxy : burp suite : caido : mitmproxy

--> chunk the incoming request / response info and send to an LLM

--> depending on the prompt you will get back different answers.

# challenges

the context length of an LLM -> you * will * have to chunk data

the LLM will miss important info that get split apart by the chunks

the speed and volume of data that passes through can be unwieldy and presents challenges.
    -> do you filter out bits before asking LLMs... if yes: which bits to filter out


# bug finder prototype



::

Using caido as my browser forwarding proxy:

I set up a "workflow" to log request and response info to a text file.

I then have a node js script that monitors this file... a bit like using "tail -f"

The incoming stream of text is split into chunks that correspond to the context length of the LLM i'm using.

For each chunk 1 or more questions for the LLM are added to a queue.

for each job in the queue the chunk + question are sent to replicate.com / llama LLM with 70b context.

The results are returned.

:: there is a filtering and processing step here to only show confident or interesting findings.

These findings can be chunked and the process can happen again to keep filtering down the findings.

...

a final "output" is created that in theory has saved you time and shows things worthy of investigation.


:::

benefits of this approach are:

    No easily detectable scanning or probing is happening. -> passive and largely undetectable from the website owners perspective
    Your accessing web pages like a normal website user would. No excessive requests are being made that would trigger a WAF or bot defence.

    ::

    The findings from the bug finder LLM tool will help target yout time and efforts instead of having to throw the kitchen sink / spray the entire website




 # passive caido workflow


```js


/**
 * @param {HttpInput} input
 * @param {SDK} sdk
 * @returns {MaybePromise<Data | undefined>}
 */
export async function run({ request, response }, sdk) {
  if (response) {
    sdk.console.log("STARTINFO")
    sdk.console.log(`  ID: ${response.getId()}`);
    sdk.console.log(`  Code: ${response.getCode()}`);
    sdk.console.log(`  Headers: ${JSON.stringify(response.getHeaders(), void 0, 2)}`);
    sdk.console.log(response.getHeaders()['Content-Type']+"......")
    var responseBody = response.getBody()?.toText()
    if (responseBody) {
      if (!response.getHeaders()['Content-Type'].includes('image') && !response.getHeaders()['Content-Type'].includes('application')) //sdk.console.log(`Response Body: ${responseBody}`);
    }
    sdk.console.log("ENDINFO")

  }
}


```

```js
/**
 * @param {HttpInput} input
 * @param {SDK} sdk
 * @returns {MaybePromise<Data | undefined>}
 */
export async function run({ request, response }, sdk) {
  if (response) {
    sdk.console.log("STARTINFO")
    sdk.console.log(`ID: ${response.getId()}`);
    sdk.console.log(`Code: ${response.getCode()}`);
    sdk.console.log(`Headers: ${JSON.stringify(response.getHeaders(), void 0, 2)}`);
    sdk.console.log(response.getHeaders()['Content-Type']+"......")
    var responseBody = response.getBody()?.toText()
    if (responseBody) {
      if (!response.getHeaders()['Content-Type'].includes('image') && !response.getHeaders()['Content-Type'].includes('application')) sdk.console.log(`Response Body: ${responseBody}`);
    }
    sdk.console.log("ENDINFO")

  }
}
```


```js


// export REPLICATE_API_TOKEN=r8
// tail -f /home/smm/.local/share/caido/logs/logging.2024-06-25.log | node watch2.js

const Replicate = require("replicate");
const crypto = require('crypto');

const replicate = new Replicate({
  auth: process.env.REPLICATE_API_TOKEN,
});

// Function to handle the chunk of text between STARTINFO and ENDINFO
async function processChunk(chunk) {
    const input = {
        top_p: 1,
        prompt: "im looking for misconfiguration, API endpoints, interesting parameters and oddities. " + chunk,
        temperature: 0.5,
        max_new_tokens: 500,
        min_new_tokens: -1
    };

    for await (const event of replicate.stream("meta/llama-2-70b-chat", { input })) {
        process.stdout.write(`${event}`)
    };
}

// Initialize an empty queue
let queue = [];
let isProcessing = false;

// Worker function to process chunks from the queue
async function worker() {
  if (!queue.length || isProcessing) return;
  isProcessing = true;

  const chunk = queue.shift(); // Take the next chunk from the queue
  await processChunk(chunk);

  isProcessing = false;
  worker(); // Process the next chunk
}

// Read from stdin
process.stdin.setEncoding('utf8');
let inputData = '';

process.stdin.on('readable', () => {
  let chunk;
  while ((chunk = process.stdin.read())!== null) {
    inputData += chunk;
  }

  const startIndex = inputData.indexOf("STARTINFO");
  const endIndex = inputData.indexOf("ENDINFO");

  if (startIndex!== -1 && endIndex!== -1 && endIndex > startIndex) {
    const chunk = inputData.substring(startIndex, endIndex).trim();
    queue.push(chunk); // Push the chunk into the queue
    inputData = ''; // Reset inputData after extracting the chunk
    worker(); // Start processing if not already running
  }
});

process.stdin.on('end', () => {
  console.log('Input stream ended.');
});


```





```
  2024-06-25T20:48:49.766085Z  INFO executor:0|arbiter:6 js|sdk: Request Body: GET / HTTP/1.1
Host: duckduckgo.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate, br
DNT: 1
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
If-None-Match: "667b067b-4d67"


2024-06-25T20:48:49.766090Z  INFO executor:0|arbiter:6 js|sdk:
 The provided log output appears to be a portion of an HTTP request and response. Here are some observations and potential issues:

1. The request method is GET, but the request body contains a POST request. This could indicate a misconfiguration or an attempt to exploit a vulnerability.
2. The User-Agent header contains a Firefox user agent, but the Accept header includes image/avif and image/webp, which are not typically supported by Firefox. This could indicate a misconfiguration or a request forgery attempt.
3. The Accept-Encoding header includes br, which is not a valid encoding. This could indicate a misconfiguration or a request forgery attempt.
4. The Upgrade-Insecure-Requests header is set to 1, which could potentially expose the client to man-in-the-middle attacks if the request is not properly validated.
5. The Sec-Fetch-Dest header is set to document, which could potentially expose the client to cross-site scripting (XSS) attacks if the request is not properly validated.
6. The Sec-Fetch-Mode header is set to navigate, which could potentially expose the client to navigational attacks if the request is not properly validated.
7. The If-None-Match header includes a hash that does not match the request method or URL, which could potentially indicate a caching issue or a request forgery attempt.
8. The Host header does not match the domain name in the URL, which could potentially indicate a DNS spoofing attack or a request forgery attempt.

```





<img src="/assistant.png" />

<img src="/assistant_answer.png" />

<img src="/caido-main.png" />

<img src="/caido.png" />

<img src="/replicate.png" />

<img src="/replicate_node.png" />

<img src="/replicate_prompt.png" />

<img src="/workflow.png" />



























