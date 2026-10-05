# Customer Feedback & Complaint Handler

An AI-powered automation built with **n8n Cloud** and **Google Gemini** that reads every customer feedback submission the moment it arrives, decides whether it is a complaint, logs it, alerts the team, and replies to the customer automatically.

**Author:** Binal Desai

---

## Problem

Customer feedback often sits unread in a form or inbox. When a customer is unhappy, a slow response can mean losing them for good. This workflow removes the manual step: every submission is analysed and answered immediately.

## What it does

1. A customer submits the feedback form (name, email, rating 1-5, feedback text).
2. **Google Gemini** returns the sentiment (positive / neutral / negative), a complaint flag (yes / no) and a one-line summary.
3. The submission is appended as a new row in **Google Sheets**.
4. A JavaScript node decides if it is a complaint and generates a discount code.
5. **If complaint:** an apology email with a discount code is sent via **Gmail**, and a **Slack** alert is posted to the team.
6. **If not a complaint:** a short thank-you email is sent.

[Workflow](docs/images/workflow.png)<img width="1953" height="724" alt="image" src="https://github.com/user-attachments/assets/4a8f71d4-eb4e-4be8-91ba-6025df38adf5" />


## Complaint rule

A submission is a complaint if **either** is true:

- Gemini judges the text to be a complaint, **or**
- the star rating is **2 or lower** (even if the text sounds positive).

| Condition | Result | Customer receives |
|---|---|---|
| Gemini says complaint OR rating <= 2 | Complaint | Apology email + discount code, Slack alert to team |
| Gemini says not a complaint AND rating >= 3 | Not a complaint | Thank-you email |

## Tech stack

| Tool | Purpose |
|---|---|
| n8n Cloud | Workflow orchestration |
| n8n Form Trigger | Public feedback form |
| Google Gemini | Sentiment and complaint detection |
| Google Sheets | Feedback log |
| Slack | Internal complaint alerts |
| Gmail | Automatic customer replies |



## Setup

### Prerequisites

- n8n Cloud account (free tier works)
- Google account (Sheets + Gmail)
- Google Gemini API key
- Slack workspace with a bot that can post to your alert channel

### Steps

1. **Import the workflow.** In n8n: *Workflows -> Import from File* and select `workflow/Customer_Feedback_and_Complaint_Handler.json`.
2. **Create the Google Sheet.** Make a sheet with these headers in row 1:
   `Name | Email | Rating | Feedback | Sentiment | IsComplaint | Summary | Timestamp`
3. **Connect credentials** in n8n and assign them to the nodes:
   - Google Gemini (PaLM) API -> *Google Gemini Chat Model*
   - Google Sheets OAuth2 -> *Append row in sheet* (then re-select your own spreadsheet)
   - Gmail OAuth2 -> *Send a message1* and *Send a message2*
   - Slack API -> *Send a message* (set your own channel and invite the bot with `/invite @YourBotName`)
4. **Activate** the workflow (toggle to *Active*).
5. Open the form's **Production URL** (not the Test URL) and submit feedback.

## Sample results

| Input | Rating | AI result | Outcome |
|---|---|---|---|
| "Worst experience ever. Package arrived damaged..." | 1 | Negative / Complaint | Slack alert + apology email |
| "Loved the product! Fast shipping." | 5 | Positive / Not a complaint | Thank-you email |
| "It was okay, nothing special." | 3 | Neutral / Not a complaint | Thank-you email |
| "Great product, love it! Sorry, I clicked the wrong star." | 1 | Positive text, complaint by rating rule | Slack alert + apology email |

## Known limitations / ideas for improvement

- The Code node generates a random `discountCode`, but the apology email template currently has `SORRY10` written in. To use the random code, replace it in the Gmail node with `{{ $json.discountCode }}`.
- Add error handling for cases where Gemini returns invalid JSON.
- Add a dashboard on top of the Google Sheet.
- Add more notification channels (e.g. Microsoft Teams, WhatsApp).

## Documentation

- [User Guide (PDF)] https://github.com/BinalDesai/Customer-Feedback-and-Complain-Handler-AI-Agent/blob/main/Customer_Feedback_Handler_User_Guide.pdf



