
# bug bounty hunting with LLMs

# about me

Stephen Monslow

* alias: m-onz / monz

* website: https://m-onz.net
* github: https://github.com/m-onz

* I work in the analytics and tagging team within NLE / IA.
* I like infosec/cyber, web application security and AI.
* currently studying to become a certified burp suite practioner 
* I'm a software developer, engineer and sometimes a mad scientist. 
* I've always been interested in infosec but never employed as my main job.
* I like programming, distributed systems, linux, BSD & much more.

# some projects I've built in the past

* turning target system controlled by a webpage from any device. Attempted to add bullet tracking using computer vision.
* drone detection systems using computer vision and machine listening
* a p2p data privacy platform including crypto architecture

# some of my accomplishments 

* became the lead front engineer for an aviation software company
* fixed a horrendous micro-service architecture for a fintech company
* fixed a serverless architecture for a linkedin competitor

# In my spare time:

* I'm part of the algorave scene as an audio visual artist and VJ
* I run an algorithmic music event and make visuals using open source software: puredata.info
* I also make music the old fashion way and play many instruments including the drums and sax.

# Bug bounty hunting

* So far I am a very unsuccessful bug bounty hunter using it to learn and understand web application security.
* I will tell all the ways I've been unsuccessful and hopefully it will help you avoid my mistakes!

I am hopefully turning a corner this year and on the brink of significantly improving my chances of getting bounties.

# bug bounty hunting and VDP

Bug bounty hunting as part of VDP: vulnerability dislosure programs.

Sometimes called "crowd sourced" security.

I think it started as safe haven for ethical hackers to report security findings but has now turned into a vibrant eco-system & booming business.

There are bug bounty platforms that act as the mediators between companies and hackers.

bug bounties are awarded to those who find serious issues.

# some bug bounty platforms

* https://www.hackerone.com/
* https://www.bugcrowd.com/
* https://www.yeswehack.com/
* https://www.openbugbounty.org/

There are public and private programs

# types of programs

* private
* public

private programs are exclusive and only selected people get to do them.

public programs are open to pretty much anybody.

# the bug bounty attack surface

* web <-- this is all I focus on.
* binary
* mobile
* source code review

# differences between pentesting and bug bounty hunting

# pentesting

-> low hanging fruit exists
-> scanners can be useful
-> can access developers to ask questions
-> can sometimes audit source code
-> no other attackers are usually competing with you
-> the application has not been audited by experts so more bugs and issues likely exist

# bug bounty hunting

-> real targets with real defence
-> no low hanging fruit
-> scanners don't reveal much
-> hardened targets that may have gone through private programs
-> huge competition globally and from seasoned full time experts

# bug bounty approaches

# pentesting approach 

-> pick a target and use common tools: burp suite / etc / http proxy / browser forwarding proxy
follow a web application pentesting methodology.

# cli driven lateral first scanning -> tomnomnom -> etc

A lateral approach where you scan many targets not just a single one.

* see [tomnomnom](https://github.com/tomnomnom/meg)
* see nuclei https://github.com/projectdiscovery/nuclei

# trends in bug bounty hunting

* targeting api's (IDOR)
* asset discovery, nuclei
* automation, use of AI
* monitor applications for changes (active monitoring)

# AI powered approach 

The subject of this talk.. similar to cli driven bug bounty hunting. Utilising automation.

# the strategy 2.0 I'm working on

I have a critera for applications:

* multiple user accounts
* complexity
* newness
* size

Test applications that are complex, have multiple user types and APIs

# vulnerabilities I'm looking for

* web cache poisening and http request smuggling
* IDOR - insecure direct object reference
* application & architecture issues
* broken access control

# LLMS / chat GPT

* You know of chatGPT and AI hype?
* chat bots.. predicting the next word on steroids
* emergent capabilities -> the abillity to compute
* can handle inconsistent input -> but always gives inconsistent output

# what are LLMs

* garbage in / garbage out generators
* you must eye ball and verify the output

# there strengths and weakness

hallucination or bulls***?

# web bug bounty hunting using LLMs

My latest efforts...

# finding targets using LLMs to filter

https://raw.githubusercontent.com/arkadiyt/bounty-targets-data/main/data/domains.txt

```
api.clearpay.com
clearpay.co.uk
clearpay.com
developers.afterpay.com
afterpay.com
mobileapi.afterpay.com
mobileapi.clearpay.com
portal.afterpay.com
portal.clearpay.co.uk
portal.clearpay.com
portalapi.eu.clearpay.co.uk
portalapi.us.afterpay.com
start.1password.com
bugbounty-ctf.1password.com
aiven.io
api.aiven.io
console.aiven.io
```

# finding targets using LLM

```js

var domains = "https://raw.githubusercontent.com/arkadiyt/bounty-targets-data/main/data/domains.txt"

const axios = require('axios');
const fs = require('fs');
const Replicate = require('replicate');

const logFilePath = './logfile_text.txt'

// Function to download the text file
async function downloadFile(url, outputPath) {
  const response = await axios({
    url,
    method: 'GET',
    responseType: 'stream',
  });

  return new Promise((resolve, reject) => {
    const writer = fs.createWriteStream(outputPath);
    response.data.pipe(writer);
    writer.on('finish', resolve);
    writer.on('error', reject);
  });
}

// Function to chunk URLs into 1024 MB chunks
function chunkUrls(urls, chunkSize) {
  const chunks = [];
  let currentChunk = [];
  let currentSize = 0;

  for (const url of urls) {
    const urlSize = Buffer.byteLength(url, 'utf8');
    if (currentSize + urlSize > chunkSize) {
      chunks.push(currentChunk);
      currentChunk = [];
      currentSize = 0;
    }
    currentChunk.push(url);
    currentSize += urlSize;
  }

  if (currentChunk.length > 0) {
    chunks.push(currentChunk);
  }

  return chunks;
}

// Function to call Replicate API
async function callReplicateAPI(chunk) {
  const replicate = new Replicate({
    auth: process.env.REPLICATE_API_TOKEN,
  });

  const input = {
    top_k: 0,
    top_p: 0.9,
    prompt: `which of these URLs is likely to allow public sign up - ideally with multiple account type... please return the URL without any superfluous information!!! \n\n${chunk.join('\n')}`,
    max_tokens: 1024,
    min_tokens: 0,
    temperature: 0.6,
    system_prompt: "You are a helpful assistant that follows instructions and does not provide text explanation.. only info requested.",
    length_penalty: 1,
    stop_sequences: "<|end_of_text|>,<|eot_id|>",
    prompt_template: "<|begin_of_text|><|start_header_id|>system<|end_header_id|>\n\nYou are a helpful assistant<|eot_id|><|start_header_id|>user<|end_header_id|>\n\n{prompt}<|eot_id|><|start_header_id|>assistant<|end_header_id|>\n\n",
    presence_penalty: 1.15,
    log_performance_metrics: false
  };

  for await (const event of replicate.stream("meta/meta-llama-3-70b-instruct", { input })) {
    process.stdout.write(event.toString());
    fs.appendFileSync(logFilePath, event.toString());
  }
}

// Main function to execute the steps
async function main() {
  const fileUrl = domains;
  const outputPath = 'urls.txt';

  // Step 1: Download the text file
  await downloadFile(fileUrl, outputPath);

  // Step 2: Read the file and chunk the URLs
  const fileContent = fs.readFileSync(outputPath, 'utf8');
  const urls = fileContent.split('\n').filter(Boolean);
  const chunkSize = 8000; // 1024 MB
  const urlChunks = chunkUrls(urls, chunkSize);

  // Step 3: Call the Replicate API for each chunk
  for (const chunk of urlChunks) {
    await callReplicateAPI(chunk);
  }
}

// Execute the main function
main().catch(console.error);
```

# finding bugs in applications

Once you have discovered a target that fits your criteria...

Using a browser forwarding proxy : burp suite : caido : mitmproxy
 chunk the incoming request / response info and send to an LLM

depending on the prompt you will get back different answers.

# challenges

the context length of an LLM -> you * will * have to chunk data

the LLM will miss important info that get split apart by the chunks

the speed and volume of data that passes through can be unwieldy and presents challenges.
    -> do you filter out bits before asking LLMs... if yes: which bits to filter out

# caido 

A lightweight rust application... run from the cli. Then access via a web page locally or from a remote VPS over SSH: )

<img src="/caido.png" />

<img src="/caido-main.png" />

# caido built in AI assistant

<img src="/assistant.png" />

<img src="/assistant_answer.png" />

# replicate

<img src="/replicate.png" />

<img src="/replicate_node.png" />

<img src="/replicate_prompt.png" />

# bug finder prototype

# caido worflow

<img src="/workflow.png" />

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
# bug bounty hunting using LLMs

the latest effotr...

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

# output from the tool...

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
# ...

# Alternative implementations of this basic idea:

These are other valid alternative implementations...

* mitmproxy -> python adapter -> send to replicate.com / or custom self hosted LLMs
* burp suite -> burp suite extension -> send to replicaite / or custom self hosted LLMs

# challenges with self hosting LLMs

* Using replicate.com allows me to use a much larger LLM (70+ billion parameters) and get back a timely response.
* Self hosting LLMs or running on personal computers is more private but the latency is greater. 
* Even a 7 billion parameter on CPU or midrange GPU can take minutes to respond... this is untennable.

caido has a client / server model that allows me to run it on a remote VPS. So for future versions I could 
 run my own machine learning models using [ollama.ai](https://ollama.com/). The proxying and AI runs on the
 remote vPS and the caido interface is accessed over SSH...

```sh
ssh -L <local port>:127.0.0.1:8080 <username>@<your vps IP address>
```

# Conclusion

I am finally making head way and I hope to refine and improve my tool and release it as an open source project.


























