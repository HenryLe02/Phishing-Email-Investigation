# Phishing Email Investigation — Two Case Studies

**Hands-on email investigation portfolio | LetsDefend training labs**

> **Training environment:** These are guided LetsDefend exercises, not live incidents. All findings below are based on the lab evidence. No suspicious links were visited and no extracted executable was run.

## Contents

- [Project overview](#project-overview)
- [Tools and approach](#tools-and-approach)
- [Case 01 — PayPal Phishing Email Investigation](#case-01--paypal-phishing-email-investigation)
- [Case 02 — Suspicious Business Email & Malicious Attachment Investigation](#case-02--suspicious-business-email--malicious-attachment-investigation)
- [Combined findings and reflection](#combined-findings-and-reflection)

---

This report brings together two independent phishing investigations. I kept the evidence and detailed analysis from both cases because the aim is to show how I reached each conclusion, not just the final answers.

## Project overview

I investigated two suspicious emails using Mozilla Thunderbird and the evidence supplied in the LetsDefend lab. The first case involved a PayPal-themed reward message and an external HTML link. The second involved a business payment dispute and a Windows executable attachment. I treated each as a separate investigation, documented the observations, and explained the limits of what I could verify.

| Case | Investigation focus |
| --- | --- |
| Case 01 — PayPal-themed phishing | Brand impersonation, message headers, SPF interpretation, external HTML link |
| Case 02 — Business email and attachment | Received header, IP reputation, ZIP extraction, SHA-256, VirusTotal file analysis |

## Tools and approach

Thunderbird (message and raw source), Windows File Explorer, HashMyFiles, and VirusTotal. I first reviewed the visible message, then checked technical indicators, and finally assessed the evidence as a whole. I did not click suspicious links or execute the extracted attachment.

Scope: These are training exercises, not production incidents. Where a source document records an unverified point, it remains unverified in this combined report.

---

# Case 01 — PayPal Phishing Email Investigation

A step-by-step investigation of a suspicious German-language email | LetsDefend training challenge

Case type: Simulated phishing email analysis • Evidence source: paypal.eml • Case status: Investigation completed; external reputation checks not performed

## 1. Why I investigated this email

In this LetsDefend challenge, I was given a password-protected ZIP archive containing an email file named paypal.eml. The message appeared to come from PayPal and told the recipient, in German, that a reward was waiting to be confirmed. My aim was to check whether the message was genuinely associated with PayPal, identify the infrastructure and links it used, and explain the evidence behind my conclusion. This is a training case, not an incident from a live organisation.

I opened the supplied email in Mozilla Thunderbird and reviewed both the visible message and its raw source. I did not click the links or open the linked HTML page.

## 2. First look: what the recipient would see

<img width="855" height="689" alt="image" src="https://github.com/user-attachments/assets/fd3960e2-2cae-457d-b212-67ac2bf60764" />


*Evidence — The message as displayed in Thunderbird, including sender, subject, PayPal branding and call-to-action button.*

The display name uses decorative characters around “P.A.Y.P.A.L”, and the body shows a large “Paypal.com” heading. At first glance, those details are meant to make the message feel official. The actual sender address, however, ends in kodehexa.net rather than a PayPal-owned domain. That difference was the first reason I questioned the message.

The subject, “Wir haben Sie angerufen, Sie haben nicht geantwortet”, means approximately “We called you, but you did not answer.” Yet the body talks about a PayPal reward and asks the recipient to confirm delivery. The subject and message do not fit together naturally. The body also contains awkward German, a customer number, the recipient’s name and email address, and a “PayPal-Guthabenkarte 1000” reward. Personal details can make a scam look credible, but their presence does not prove the sender is genuine.

The prominent button says “Setzen Sie die Lieferung fort” (“Continue the delivery”). It gives the recipient a clear next action without explaining why a PayPal reward requires a separate delivery-confirmation page. This combination of brand imitation, an unexpected reward and a prompt to act is consistent with a phishing lure.

## 3. Opening the raw email headers

To check what was behind the display name, I opened Thunderbird’s Message Source view (Ctrl + U). Email headers record delivery information that is not normally visible in the message window. I looked for the From address, Return-Path, Received lines and authentication results. These fields help distinguish what the email claims from what the receiving mail system actually observed.

<img width="855" height="336" alt="image" src="https://github.com/user-attachments/assets/f593675e-04f0-4be5-b742-be62712a740b" />


*Evidence — Message Source: Return-Path, Received chain, Received-SPF and sender information.*

| Header field | Observed value |
| --- | --- |
| Visible From | Decorative PayPal display name; address at kodehexa.net |
| Return-Path | bounce@rjttznyzjjzydnillquh.designclub.uk.com |
| Delivered-To | krystalia@gmail.com |
| Receiving server | mx.google.com |
| SPF-associated connecting IP | 134.195.196.43 |
| Additional Received hop | efianalytics.com (216.244.76.116) |

## 4. What the different sender fields tell us

### 4.1 From and Return-Path are not the same thing

The From field is the identity presented to the recipient. In this case it uses a PayPal-like display name, while the underlying address belongs to kodehexa.net. The Return-Path is the envelope sender used for delivery handling and bounced messages. Here, it is bounce@rjttznyzjjzydnillquh.designclub.uk.com. A different Return-Path is not automatically malicious—legitimate mailing systems often use one—but the combination of unrelated domains and PayPal branding makes the claimed identity difficult to trust.

<img width="855" height="313" alt="image" src="https://github.com/user-attachments/assets/e5277160-3f1c-473c-a245-0adb777335be" />


<img width="855" height="336" alt="image" src="https://github.com/user-attachments/assets/571aac6c-9618-43a4-9013-4c4901f537e7" />


<img width="855" height="335" alt="image" src="https://github.com/user-attachments/assets/f98ee8e7-261d-4632-a083-5dc245e45d1f" />


*Evidence — Additional header fields show Received-SPF, From, X-EMAIL and Envelope-To.*

The raw source also contains X-EMAIL and unsubscribe-related fields. These are useful observations, but they do not establish legitimacy on their own. I did not assume the address in an X- header was a verified sender identity.

### 4.2 Reading the Received chain

The Received headers describe mail-server hops. One visible line shows Google receiving the message from a host identifying itself as foresthillrestaurant.com, with the connecting host recorded as capchrist.org [134.195.196.43]. Another line refers to efianalytics.com [216.244.76.116]. I recorded both addresses because they may help further infrastructure analysis. They should not automatically be described as the attacker’s personal IP address: mail may pass through multiple servers, and header trust depends on which server added each line.

<img width="855" height="125" alt="image" src="https://github.com/user-attachments/assets/8caebcf5-3097-424d-8901-12bc87320894" />


*Evidence — Received-SPF and Authentication-Results alongside the receiving Google server information.*

## 5. Why SPF passed even though the message looked suspicious

The Received-SPF header reports “pass”. The accompanying text states that the domain used in the envelope sender authorised 134.195.196.43 to send mail. This is an important distinction: SPF checks whether a server is authorised for the envelope-sender domain; it does not prove that the display name is truthful or that the email came from PayPal.

<img width="855" height="123" alt="image" src="https://github.com/user-attachments/assets/cc77658c-8a5f-45f3-b0ab-9a36906851be" />


*Evidence — Authentication result and Return-Path shown together, making the domain checked by SPF visible.*

The screenshot also shows ARC-Authentication-Results reporting spf=pass. ARC records authentication information as a message moves through mail systems; seeing an ARC header is not the same as verifying that PayPal authorised this message. I did not find sufficient visible evidence in the supplied screenshots to make a reliable claim about DKIM or DMARC pass/fail, so those results remain unconfirmed in this report.

## 6. Finding the suspicious URL

Next, I searched the HTML source for “https” and inspected a link embedded in the message. The visible URL points to a hosted HTML file rather than an obvious PayPal website. I did not visit it, so I cannot confirm the page contents, whether it redirects, or whether it attempts credential collection.

<img width="855" height="462" alt="image" src="https://github.com/user-attachments/assets/b7278ece-a831-49fe-bf8d-3fdda6d74d85" />


*Evidence — HTML source contains a URL beginning with https://storage.googleapis.com/... and pointing to an .html file.*

| URL component | Observation |
| --- | --- |
| Scheme | https — encrypts the connection but does not certify the site as safe |
| Hostname | storage.googleapis.com — a legitimate Google Cloud Storage hostname |
| Path | /hqyoqzatqthj/aemmfcylvxeo.html — bucket/object path visible in the evidence |
| Fragment | #... — additional URL text; its purpose was not established |

Google Cloud Storage is a legitimate service. The concern is its use inside an email pretending to represent PayPal. A phishing operator can host content on a legitimate cloud platform, so a trusted platform name alone does not validate the page. The screenshot shows an embedded HTML link; it does not conclusively establish that every visible button in the email points to this same URL. Mapping each clickable element would require reviewing all relevant href attributes.

## 7. Evidence assessment

Rather than relying on one suspicious detail, I considered the message as a whole. The PayPal-like branding conflicts with the actual From domain. The Return-Path uses another unrelated domain. The subject suggests a missed call, while the body promotes a reward and delivery confirmation. The email also contains an external hosted HTML link. These indicators reinforce each other and support a phishing assessment.

| Evidence | Interpretation |
| --- | --- |
| PayPal-themed display name + kodehexa.net | Brand impersonation indicator |
| Unrelated Return-Path domain | Needs scrutiny; not malicious by itself |
| SPF = pass for envelope sender | Server authorised for that domain, not proof of PayPal authenticity |
| Reward + delivery confirmation prompt | Social-engineering incentive to click |
| Google Cloud Storage HTML URL | Suspicious in context; destination content not verified |

## 8. Verdict, limitations and recommended handling

**Final assessment: Phishing email (high confidence based on the combined content and header indicators).**

The LetsDefend challenge also marked the phishing classification as correct. This is a classification of the email, not a claim that I observed stolen credentials, malware execution or a successful compromise.

In a real security workflow, I would preserve the original .eml, record the relevant URLs and mail-server indicators, quarantine the message where appropriate, and check whether the recipient interacted with it. If interaction were confirmed, I would escalate according to the organisation’s incident process. Before recommending broad domain or IP blocks, I would check whether the infrastructure is shared and whether the indicators are specific enough to avoid disrupting legitimate traffic.

Limitations: I did not run a live VirusTotal/urlscan reputation check, inspect the hosted HTML page, verify DKIM/DMARC, or review recipient endpoint/network logs. Those tasks would be sensible next steps, but they are not presented here as completed work.

## 9. What I learned

This case helped me see why phishing analysis needs more than spotting a suspicious-looking email. I learned to separate the sender identity shown to a person from the envelope sender checked by SPF, read the mail delivery chain without assuming every IP belongs to the attacker, and assess a cloud-hosted link in context. Most importantly, I practised writing down what the evidence actually supports and being explicit about what I could not verify.

## Evidence register

Screenshots in this report were captured from the LetsDefend lab and the Thunderbird Message Source view. The original evidence was supplied as paypal.eml inside the training archive. Screenshots are illustrative evidence; they do not replace preservation of the original message file.

---

# Case 02 — Suspicious Business Email & Malicious Attachment Investigation

**LetsDefend – Email Analysis Challenge | Hands-on security investigation**

Investigation scope: Email header review, originating IP reputation, attachment inspection, SHA-256 hashing and malware reputation lookup.

## 1. Case Overview

In this lab, I investigated an email that appeared to come from a scientific equipment company. The sender said there was a problem with the bank details on a proforma invoice and asked the recipient to check an attached payment document. My task was to work out whether the email was suspicious and support my conclusion with technical evidence, rather than judging it by appearance alone.

I reviewed the message in Mozilla Thunderbird, checked the email headers, looked up the originating IP address, and examined the extracted attachment without running it. I then calculated its SHA-256 hash and searched that hash on VirusTotal.

## 2. Initial Email Review

<img width="855" height="730" alt="image" src="https://github.com/user-attachments/assets/0712367c-aec3-48b0-b43c-f4cebb49b422" />


*Figure — The message as displayed in Mozilla Thunderbird.*

| Field | Value |
| --- | --- |
| Sender | Yan Ting <yanting@united.com.sg> |
| Recipient | admin@malware-traffic-analysis.net |
| Subject | united scientific equipment |
| Date header | 08 February 2021, 23:15:11 (UTC−8) |
| Thunderbird display | 09 February 2021, 07:15 |

The message described incorrect bank details, difficulty reaching a colleague and a request to confirm the attached payment slip. I found this concerning because it used a believable business payment issue to encourage the recipient to open a file. Still, a payment-related email is not automatically malicious, so I continued with the technical checks.

## 3. Email Header Analysis

<img width="855" height="722" alt="image" src="https://github.com/user-attachments/assets/f71045c6-26f0-4cd6-a216-2dc620661224" />


*Figure — Message Source in Thunderbird showing sender, recipient, date and Received header.*

I opened the raw message source to look beyond the visible sender name. The Return-Path was yanting@united.com.sg, and Delivered-To was admin@malware-traffic-analysis.net. The Received header included the line:

    Received: from united.com.sg (unknown [71.19.248.52])

This provided an IP address to investigate. The From and Return-Path fields were consistent with each other, but that alone did not authenticate the message. The Date field showed 8 February in UTC−8, which explains why Thunderbird displayed 9 February after time-zone conversion. I did not verify SPF, DKIM or DMARC results from the evidence available in this case.

## 4. Originating IP Reputation

<img width="855" height="595" alt="image" src="https://github.com/user-attachments/assets/149c7272-66cb-4638-ac5b-e762ade723be" />


*Figure — VirusTotal IP lookup for 71.19.248.52, showing Canada and no vendor detections.*

| Field | Observed result |
| --- | --- |
| IP address | 71.19.248.52 |
| Reported location | Canada (CA) |
| ASN | AS11831 – eSecureData |
| VirusTotal detections | 0/92 |

I searched the IP address on VirusTotal. The result showed 0 out of 92 security vendors flagging it. I treated that as a reputation result, not proof that the email was safe. A server can deliver a malicious attachment even if its IP has no detections, so I moved on to the attachment itself.

## 5. Attachment Extraction and Inspection

<img width="855" height="294" alt="image" src="https://github.com/user-attachments/assets/c365755f-436a-4cd0-a3b3-5401a3622cd2" />

*Figure — The ZIP attachment in the lab Files folder.*

<img width="855" height="538" alt="image" src="https://github.com/user-attachments/assets/515e40ec-3b21-44bc-a62f-a7e3cff4e2e7" />


*Figure — The extracted folder containing the executable attachment.*

The lab provided a separate password-protected ZIP archive for the attachment. I extracted it inside the LetsDefend lab using the supplied password, infected. The resulting filename was “united scientific equipent.exe” — including the unusual spelling of “equipent”, which I kept exactly as shown in the evidence.

This was a major warning sign. The email talked about an invoice and payment slip, but the attachment was a Windows executable (.exe), not an ordinary invoice document. I did not execute the file. Instead, I used a hashing tool to identify it and check for existing threat intelligence.

## 6. Calculating the SHA-256 Hash

<img width="855" height="384" alt="image" src="https://github.com/user-attachments/assets/4fa331fa-71da-461b-9bbf-dab669d7d5dc" />


<img width="855" height="594" alt="image" src="https://github.com/user-attachments/assets/2140a8eb-ab7c-4e69-b252-e848f8a7c4fa" />


*Figure — HashMyFiles Properties showing the file hashes and metadata.*

I loaded the extracted executable into HashMyFiles and opened its Properties. SHA-256 is a cryptographic hash: it produces a 256-bit value that can be used as a fingerprint of the file’s contents. The hash is useful for checking whether the same file has been reported elsewhere, even under another filename.

**SHA-256:** `9909753bfb0ac8ab165bab3555233d03b01a9274a92e57c022f87ccbe51ca415`

A hash does not tell me by itself whether a file is malicious. It is an identifier that I can use for a reputation lookup. I searched this exact SHA-256 value on VirusTotal rather than running the executable.

## 7. VirusTotal File Analysis

<img width="855" height="836" alt="image" src="https://github.com/user-attachments/assets/2c214a52-50d5-4070-b310-61f2de4b798e" />


*Figure — VirusTotal reports 59 of 70 security vendors flagging the file as malicious.*

| Field | Finding |
| --- | --- |
| SHA-256 lookup | Matched the hash calculated in HashMyFiles |
| Detection result | 59/70 vendors flagged the file as malicious |
| File type | Windows executable (PE EXE) |
| Alternative name shown | cDisplayClass300.exe |
| Threat labels observed | Trojan-related labels, including Loki and AgentTesla classifications |

This was the strongest finding in the case. VirusTotal reported that 59 out of 70 security engines detected the file as malicious. The result also displayed a different filename, cDisplayClass300.exe. That did not invalidate the comparison, because I searched by the file’s SHA-256 hash, not by its name.

The vendor labels were not completely consistent, so I would not claim that I had proved a specific malware family. I also did not perform dynamic analysis or observe the executable running. My conclusion is based on the attachment type, the email context and the strong reputation evidence.

## 8. Final Assessment and Recommended Response

I assessed this message as a likely malware-delivery phishing attempt. The email used a plausible payment dispute to encourage the recipient to open an attachment. The attachment was an executable, and the exact file hash returned a strong malicious detection result (59/70) on VirusTotal. The clean IP reputation result (0/92) did not outweigh the file evidence.

In a real workplace, I would advise the recipient not to open or run the attachment, report the email through the organisation’s security process, preserve the original email and its headers, and escalate the file hash and related indicators for further investigation. I would also check whether other users had received the same message or file, using authorised security tools.

## 9. What I Learned

This case helped me understand why I should not rely only on how an email looks or on a single reputation score. I practised reading raw email headers, finding an originating IP, checking IP reputation, safely inspecting an attachment, calculating SHA-256 and using that hash for a VirusTotal lookup. The most useful lesson was how to bring those separate findings together into a clear, evidence-based decision.

## Scope and Limitations

This is a guided LetsDefend training investigation, not a production incident. I did not execute the attachment, confirm its exact malware family, verify the sender’s authentication alignment or establish whether any recipient was compromised. All conclusions are limited to the screenshots and results collected during the lab.

---

# Combined findings and reflection

These two cases reinforced the same lesson in different ways: a single indicator does not decide whether an email is safe. In Case 01, the email presented itself as PayPal, but the visible sender domain and Return-Path did not support that identity. SPF passed for the envelope-sender domain, which was not proof of PayPal legitimacy. The HTML source also included a link to a cloud-hosted page; I did not visit the destination, so its behaviour remains unknown.

In Case 02, the sender and Return-Path were consistent, and the originating IP returned no VirusTotal detections in the observed lookup. However, the payment-themed message led to an executable attachment. I extracted the sample without running it, calculated its SHA-256 hash, and found that 59 of 70 engines in the observed VirusTotal result flagged the file. This was much stronger evidence of risk than the IP reputation result.

## What I would do in a real environment

I would preserve the original email and relevant headers, record the specific indicators, advise the recipient not to interact with suspicious content, and follow the organisation’s escalation process. I would check whether other users received the same message and whether anyone clicked the link or executed the file, using authorised mail, endpoint and network logs. I would avoid broad blocking of shared cloud infrastructure without checking potential impact.

## Limitations

The first case did not include a live visit to the hosted page or verification of DKIM/DMARC results. The second did not include dynamic malware analysis, a confirmed malware-family attribution or evidence of endpoint compromise. I have not described any of these unperformed steps as completed work.

## Overall learning

I learned to separate what a message claims from what the headers and file evidence actually show. I also became more comfortable explaining technical findings in plain English: what I checked, why it mattered, what I found, and what I still could not confirm. That is the approach I want to bring to future cybercrime investigations.
