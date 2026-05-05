# VulnBegin Web CTF Writeup

## Subdomain Enumeration

1. I am met with the homepage located: https://quartz.ctfio.com
2. The CTF provides a OSINT domain to explore that being: vulnbegin.co.uk
3. Run Burp to see if there's anything hidden
   ![alt text](images/image-1.png)
4. I am using subfinder to scout subdomains 
   ![alt text](images/image-3.png)
5. Found these four subdomains: 
   - www.vulnbegin.co.uk (known)
   - v64hss83.vulnbegin.co.uk (new)
   - server.vulnbegin.co.uk (new)
   - www.vulnbegin.co.uk (new)
6. Plug them into our target site with different variations:
   - https://[new].quartz.ctfio.com 
   - https://[new].quartz.ctfio.com/[new]
   
   ![alt text](images/image-2.png)
   ![alt text](images/image-4.png)
7. Two keys found, one with additional information 'User Not Authenticated':
   - `[^FLAG^E858ED9649E57BECE9ACD1A4C60D3446^FLAG^]`
   - `[^FLAG^047524FE61AE6B5FD1D184994C7322FC^FLAG^]`

---

## Fuzzing - Hidden Directories 

1. Ran the command:
   ```bash
   ffuf -w content.txt -u "https://server.tanzanite.ctfio.com/FUZZ" -t 4
   ```
   - `-w`: input file wordlist
   - `-u`: target with FUZZ placeholder
   - `-t`: number of threads

   ![alt text](images/image-5.png)

2. Testing discovered endpoints:
   - https://tanzanite.ctfio.com/cpadmin/login
     ![alt text](images/image-6.png)
   - https://tanzanite.ctfio.com/robots.txt
     ![alt text](images/image-7.png)
3. Testing /secret_d1rect0ry/
   ![alt text](images/image-8.png)

---

## Brute Force 

1. Found /cpadmin which leads to a login page with username and password input fields
2. Spamming the login page doesn't produce CAPTCHA - unlimited login attempts possible. Generic error message: 'Username is invalid'
3. Created a POST request to start brute forcing
4. Moved the POST request to Intruder and set payload parameter on Username field. Loaded provided username list and set stop condition when 'Username is invalid' response is missing
   ![alt text](images/image-9.png)
5. Attack stopped at 'admin' - viewing the response:
   ![alt text](images/image-10.png)
   ![alt text](images/image-11.png)
6. Cracking the password - looking for HTTP 302 Redirect status code indicating successful login:
   ![alt text](images/image-12.png)
   - `[^FLAG^93D7491FB4B054FB5C5AC3E0292BE41C^FLAG^]`

---

## Cookie & API

**Cookie found:** `token=2eff535bd75e77b62c70ba1e4dcb2873`

1. Successfully authenticated. Now finding more directories under cpadmin using the cookie
2. Found endpoints:
   - env
   - login
   - logout
   
   ![alt text](images/image-14.png)

3. ENV files usually contain API keys:
   ![alt text](images/image-15.png)
   
   **API Key:** `X-Token: 492E64385D3779BC5F040E2B19D67742`
   - `[^FLAG^F6A691584431F9F2C29A3A2DE85A2210^FLAG^]`

4. Remember from subdomain enumeration: found server.quartz.ctfio.com with 'User not authenticated' message
5. Now with the API key, revisiting the site in Burp:
   ![alt text](images/image-16.png)
6. Send the GET request to Repeater
7. Add the API key to authenticate:
   ![alt text](images/image-17.png)
   - `[^FLAG^0BDC60CC5E283476E7107C814C18DCCF^FLAG^]`

---

## More Fuzzing

1. With the API key obtained, continuing to fuzz for more endpoints:
   ```bash
   ffuf -w content.txt -u https://server.barite.ctfio.com/FUZZ -t 4 -H "X-Token: 492E64385D3779BC5F040E2B19D67742"
   ```

2. Results found:
   - css
   - js
   - robots.txt
   - user 

3. Testing each endpoint
4. robots.txt familiar result:
   ![alt text](images/image-18.png)

5. User endpoint found:
   ![alt text](images/image-19.png)

6. Running in Burp and adding the API key:
   ![alt text](images/image-20.png)

7. Navigating to /user/27 with API key in Burp:
   ![alt text](images/image-21.png)

8. Going to /user/27/info with API key:
   ![alt text](images/image-22.png)
   - `[^FLAG^7B3A24F3368E71842ED7053CF1E51BB0^FLAG^]`

---

## Insecure Direct Object Reference (IDOR)

1. Noticed numeric ID after /user/ - potential to enumerate other user accounts
2. Automated enumeration using ffuf:
   ```bash
   ffuf -w <(seq 1 100) -H "X-Token: 492E64385D3779BC5F040E2B19D67742" -u "https://server.hennessy.ctfio.com/user/FUZZ/info" -t 4
   ```
   ![alt text](images/image-23.png)

3. Accessing /user/5/info
4. Repeating Burp request with API key:
   ![alt text](images/image-24.png)
   - `[^FLAG^3D82BE780F46EE86CE060D23E6E80639^FLAG^]`
