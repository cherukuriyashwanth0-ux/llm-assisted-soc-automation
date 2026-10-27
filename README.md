# LLM-Assisted SOC Automation: Malware Alert Triage with Splunk and Gemini

A lab project that sends a Windows Defender detection through Splunk and Zapier to a Google Gemini LLM, which writes a SOC investigation summary that is emailed to the analyst.

> **Note:** The detection uses the **EICAR test file**, a harmless antivirus test pattern. No real malware was used.

## Architecture

Windows 11 VM → Windows Defender → Splunk Universal Forwarder → Splunk Enterprise → Splunk Alert (Webhook) → Zapier Catch Hook → Google Gemini → Gmail

## Environment

| Component | Details |
|---|---|
| Host | MacBook Air M1, 8 GB RAM |
| Virtualization | UTM 4.7.5 |
| Endpoint | Windows 11 VM (`WIN11-VM-UTM`) with Windows Defender |
| SIEM | Splunk Enterprise 10.4.2 (macOS) |
| Log forwarding | Splunk Universal Forwarder |
| Automation | Zapier (Webhooks by Zapier, Catch Hook) |
| LLM | Google Gemini 2.5 Flash (via Zapier) |
| Notification | Gmail |

## Workflow

1. **Detection:** Defender blocks the EICAR file and logs Event ID `1116` (`Virus:DOS/EICAR_Test_File`, Severe).
2. **Ingestion:** The Universal Forwarder sends the Defender Operational log to Splunk.
3. **Alert:** A Splunk alert searches for EventCode 1116 containing `EICAR_Test_File` and triggers a webhook.
4. **Automation:** Zapier receives the alert through a Catch Hook.
5. **LLM analysis:** Gemini receives the alert evidence and produces a structured SOC summary.
6. **Notification:** Gmail sends the alert evidence and the Gemini analysis to the SOC analyst.

### Splunk detection query

```spl
index=main source="WinEventLog:Microsoft-Windows-Windows Defender/Operational" EventCode=1116
| search Message="*EICAR_Test_File*"
```

## Gemini output sections

Alert summary, host and event ID, detection evidence, severity and confidence (with uncertainty), recommended investigation steps, and recommended containment actions.

## Evidence

### 1. Windows Defender detection
![Defender Event](<WIN11-VM-UTM Windows Defender Event.png>)

### 2. Splunk log investigation
[View screenshot](<Splunk Log Investigation.tiff>)

### 3. Splunk alert configuration
![Splunk Alert](<Splunk Alert .png>)

### 4. Zapier Catch Hook receiving the alert
![Zapier Catch Hook](<Zapier Catch Hook.png>)

### 5. Zapier Gemini integration
![Zapier Gemini](<Zapier Google AI Integration.png>)

### 6. Zapier email step
![Zapier Email](<Zapier Email .png>)

### 7. SOC email received
![Email 1](<Email Notification1.PNG>)
![Email 2](<Email Notification2.PNG>)
![Email 3](<Email Notification3.PNG>)

## Limitations

- Each step was verified individually. The pipeline was not left running continuously.
- Gemini only provides analysis and **recommendations**. No containment action is executed automatically.
- The EICAR test events had timestamps that did not fit a normal "last 24 hours" search window, so the alert was tested with an "All time" range.
- The LLM output is not validated and should be reviewed by a human analyst.

## Security note

The Zapier webhook URL and API keys are not included in this repository.
