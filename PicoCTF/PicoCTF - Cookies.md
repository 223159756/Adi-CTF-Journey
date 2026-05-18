# Cookies

| Field      | Detail     |
| ---------- | ---------- |
| Platform   | PicoCTF    |
| Category   | Web        |
| Difficulty | Easy       |
| Points     | 40         |
| Date       | 2026-05-18 |
| Status     | Solved     |

---

## Description

![challenge description](../Screenshots/Pasted%20image%2020260518140920.png)

---

## Initial Observations

A web app with a form that takes a cookie name and returns a colour response based on validity. The name says "Cookies" so the HTTP cookie is the obvious target. Opened Burp straight away.

---

## Approach

**Step 1 -** Loaded the page and intercepted the form submission in Burp. The request had a cookie `name=-1` , clearly a default value. The server was using this cookie to track state.

![homepage form](../Screenshots/Pasted%20image%2020260518151451.png)

![intercepted request showing name=-1](../Screenshots/Pasted%20image%2020260518151357.png)

**Step 2 -** Sent to Repeater, switched to GET `/check` (POST returned 405). Started incrementing `name=0`, `name=1` etc. The server returned different cookie flavours for each valid number but always reset the cookie back if sent to `/`.

![Repeater GET /check](../Screenshots/Pasted%20image%2020260518151551.png)

![valid cookie responses](../Screenshots/Pasted%20image%2020260518151711.png)

**Step 3 -** Sent to Intruder, set `name` as the payload position, ran numbers 0-28 as a Sniper attack. Sorted by length; every response was around 2050 bytes except `name=18` which came back at 1360. which seemed suspicious.

![Intruder setup](../Screenshots/Pasted%20image%2020260518151926.png)

![Intruder results - name=18 outlier](../Screenshots/Pasted%20image%2020260518151958.png)

![name=18 flag response](../Screenshots/Pasted%20image%2020260518152057.png)

![flag](../Screenshots/Pasted%20image%2020260518152852.png)

**Step 4 -** Back in the browser, set `name=18` in the cookie via dev tools and visited `/check`. Flag on the page.

![browser cookie set to 18](../Screenshots/Pasted%20image%2020260518155426.png)

![flag in browser](../Screenshots/Pasted%20image%2020260518155445.png)

**Flag: `picoCTF{3v3ry1_l0v3s_c00k135_a4dadb49}`**

---

## Tools Used

- **Burp Suite Proxy** - intercept and inspect HTTP requests
- **Burp Repeater** - manually modify and resend requests
- **Burp Intruder** - brute force cookie values 0-28, sort by response length to find the outlier
- **Inspect Element** - Using applications to change cookie value and test render for flags

---

## Key Finding

Response length was the tell. Every valid cookie value returned ~2050 bytes, `name=18` returned 1360. looking at outlier resulted in the flag.

---

## Lessons Learned

**1. Sort Intruder results by length**
All responses returned 200 here. Length was the only differentiator, sort by length first when brute forcing web parameters.

**2. Repeater before Intruder**
Manually confirming the right endpoint and request format in Repeater before running Intruder saved time.
