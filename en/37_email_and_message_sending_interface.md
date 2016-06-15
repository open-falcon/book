##Email and Message Sending Interface

This component has no code and each company needs to provide code by itself.

The monitoring system needs to send alarm emails or messages after alarm events occur. Each company may have its own email server and sending method and its own message channel and sending method. To adapt to different companies, falcon sets up a specification for the access scheme where each company needs to provide message and email sending http interfaces.

Short message sending http interface:

```
method: post
params:
  - content: Short message content
  - tos: Multiple mobile phone numbers separated by commas

```
Email sending http interface:

```
method: post
params:
  - content:Email content
  - subject: Email subject
  - tos: Multiple email addresses separated by commas

```

