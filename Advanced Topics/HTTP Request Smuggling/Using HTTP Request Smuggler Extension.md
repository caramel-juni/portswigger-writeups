

Convert a request to chunked:
![](attachments/48981.png)

Then can launch automatic TE.CL/CL.TE attacks:
![](attachments/12507.png)

Ensure to, when working with requests:
#### Setup:
- Show non-printable characters -> `ON`
- Update Content-Length automatically -> `OFF`
- Switch to HTTP 1.1
- Send a sanity check `POST` to base endpoint `/`


##### TurboIntruder for `TL.CE`:
*For all TE.CL vulnerabilities, **USE HTTP REQUEST SMUGGLER'S TURBO INTRUDER***. To do so:
- Build your attacker request in Repeater (like we have above)
- Right click, and choose `Extensions` - `HTTP Request Smuggler` - `TE.CL smuggling attack`.
  ![](attachments/Pasted%20image%2020260615163113.png)
   This will **open a Turbo Intruder window to send requests repeatedly**, at a **fast enough speed**, to attempt to **reach and poison the same backend server**.
- Edit the code to remove the pre-filled contents of the "prefix" request, as we've built ours manually and are **here just for TurboIntruder's rapid sequential sending speed**:
  ![](attachments/Pasted%20image%2020260615163340.png)
- Then, press Attack, and review the requests to ensure the content being sent is intended. It should now only be sending them in rapid succession (save a few minor tweaks which didn't seem to impede testing).
I could not for the life of me get these to work with either regular Intruder, manual sending, group based sending of requests, etc. - **but this worked.** 
So, TLDR; `TE.CL` combos are finnicky and require precise timing & tools to exploit.
