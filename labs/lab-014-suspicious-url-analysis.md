# Lab 014 - Suspicious URL Analysis

## Executive Summary

This lab continued the investigation of `email3.eml` from Lab 013. The analysis focused specifically on embedded URLs, tracking parameters, remote images, and redirect behaviour.

Multiple tokenized URLs were identified under the `t.teckbe.com` domain. The email also contained externally hosted images and a hidden zero-size tracking pixel. These findings confirmed the use of email campaign tracking mechanisms, although the available evidence did not confirm credential theft, malware delivery, or a malicious landing page.

## Objective

The objective was to identify and analyse URLs embedded in the email without opening or directly visiting them.

## Environment / Data Source

Host: TryHackMe training environment  
Tool: Mozilla Thunderbird email source viewer  
Data source: `email3.eml`  
Content type: Quoted-printable HTML email  

## Observed Activity

The email contained multiple URLs using the following domains:

- `t.teckbe.com`
- `img.teckbe.com`

Different URLs were associated with unsubscribe actions, a report-spam action, promotional images, the main call-to-action button, and email-open tracking.

## Evidence

### Evidence Screenshot

![Suspicious URL analysis](01-url-analysis.png)

The raw HTML source of `email3.eml` showed tokenized links under `t.teckbe.com`, externally hosted images under `img.teckbe.com`, and a zero-size remote image consistent with email-open tracking.

### Main Call-to-Action URL

```text
http://t.teckbe.com/p/?j3=EOowFcEwFHl6EOAyFcoUFVTVEchwFHlUFOo6lVTTDcATE7oUE7AUET==
```

The same URL was connected to:

- the `CLICK HERE` button,
- the first promotional image,
- the second promotional image.

### Remote Image URLs

```text
http://img.teckbe.com/i/012021/161115137288_1.jpg
http://img.teckbe.com/i/012021/161115137288_2.jpg
```

These images were loaded from external infrastructure when the email content was rendered.

### Hidden Tracking Pixel

```html
<img
  src="http://t.teckbe.com/p/?j3=EOowFcEwFHl6EOAyFcoUFVTVEchwFHlUFOo6KVTTDc0="
  height="0px"
  width="0px"
>
```

The zero-size remote image was invisible to the recipient and was likely used to record when the email was opened.

### Additional URL Characteristics

- The URLs used HTTP rather than HTTPS.
- Different `j3` values were assigned to different email actions.
- The final destination of the main call-to-action was not directly visible.
- Multiple clickable elements were routed through the same tracking domain.
- The email used separate tracking tokens for clicks, unsubscribe actions, and open tracking.

## Analysis

The email used tokenized URLs under the `t.teckbe.com` domain. Different `j3` values were associated with unsubscribe actions, promotional content, the call-to-action button, and email-open tracking.

The main call-to-action URL was reused behind several clickable elements, indicating centralised click-tracking or redirection infrastructure.

The strongest finding was an image configured with `height="0px"` and `width="0px"`. This was consistent with a hidden tracking pixel. When the email client requested this remote resource, the sender could potentially record that the message was opened and collect connection-related metadata.

The URLs were opaque and did not reveal their final destinations. However, the HTML evidence alone did not confirm credential theft, malware delivery, or redirection to a malicious website.

## Risk

The tracking URLs could allow the sender to:

- confirm that the recipient opened the email,
- record clicks on email content,
- collect connection-related metadata,
- redirect users to an unknown destination.

The use of tracking infrastructure increased privacy and security concerns, but malicious intent was not conclusively established.

## Recommended Next Steps

- Analyse the domains using passive reputation services.
- Investigate the final redirect destination in a controlled sandbox.
- Compare the domains with known phishing and spam intelligence sources.
- Correlate the URLs with the sender domain and originating IP identified in Lab 013.
- Quarantine the email if organisational policy prohibits unsolicited tracking content.

## MITRE ATT&CK Mapping

No MITRE ATT&CK technique was assigned because the available evidence did not confirm credential phishing, malware execution, or another specific adversary technique.

## Final SOC Summary

The email contained multiple tokenized tracking and redirection URLs hosted under the `teckbe.com` infrastructure. The main call-to-action button and promotional images used the same opaque redirect URL, while a hidden zero-size image was likely used as an email-open tracking pixel. The evidence confirmed tracking behaviour but was insufficient to classify the message as confirmed malicious phishing. The email should remain suspicious and may require further passive reputation and redirect analysis.

## Lessons Learned

This lab demonstrated how to identify URLs within quoted-printable HTML email content and distinguish between visible links, remote images, redirect links, and hidden tracking pixels. It also reinforced the importance of separating confirmed technical behaviour from unsupported assumptions.

## Training Source

The email sample and controlled investigation environment were provided by TryHackMe. The analysis, evidence interpretation, risk assessment, and report structure were independently prepared for portfolio purposes.
