# CAPTCHA Handling Documentation

## Overview

ChatGPT Automation MCP includes automatic CAPTCHA detection and manual resolution support. The system can detect various types of CAPTCHAs and wait for the user to solve them manually.

## Supported CAPTCHA Types

The system detects the following CAPTCHA types:

### 1. Google reCAPTCHA
- `iframe[src*="recaptcha"]`
- `iframe[title*="recaptcha"]`
- `div.g-recaptcha`

### 2. hCaptcha
- `iframe[src*="hcaptcha"]`
- `div.h-captcha`

### 3. Cloudflare
- `iframe[src*="cloudflare"]`
- `div#challenge-stage`
- `div.cf-browser-verification`

### 4. OpenAI-specific Challenges
- "Just a moment"
- "Checking your browser"

### 5. Generic Security Checks

**English:**
- "verify you are human"
- "prove you are not a robot"
- "Security check"
- "Complete the security check"

**Russian:**
- "подтвердите, что вы человек"
- "докажите, что вы не робот"
- "Проверка безопасности"

## Implementation Details

### Detection Method: `_detect_captcha()`

Located in `browser_controller.py:458`

**Functionality:**
- Checks all CAPTCHA selectors on the page
- Verifies element visibility (not just presence)
- Returns `True` if any CAPTCHA is detected and visible
- Logs the specific selector that matched

**Return Values:**
- `True` - CAPTCHA detected and visible
- `False` - No CAPTCHA found

### Resolution Method: `_wait_for_captcha_resolution()`

Located in `browser_controller.py:517`

**Functionality:**
- Waits for user to manually solve CAPTCHA
- Default timeout: 30 minutes (1800 seconds)
- Checks every 5 seconds if CAPTCHA is still present
- Logs progress every 30 seconds

**Parameters:**
- `timeout: int` - Maximum wait time in seconds (default: 1800)

**Return Values:**
- `True` - CAPTCHA was solved successfully
- `False` - Timeout occurred

**User Experience:**
```
⚠️  CAPTCHA DETECTED!
📋 Please solve the CAPTCHA manually in the browser window.
⏳ Waiting up to 30 minutes for resolution...
⏳ Still waiting for CAPTCHA resolution... 1770s remaining
⏳ Still waiting for CAPTCHA resolution... 1740s remaining
...
✅ CAPTCHA solved successfully!
```

## Integration Points

CAPTCHA detection is integrated at these critical points in the authentication flow:

### 1. Initial Page Load
**Location:** `browser_controller.py:226-229`
```python
if await self._detect_captcha():
    logger.info("CAPTCHA detected on page load, waiting for resolution...")
    if not await self._wait_for_captcha_resolution():
        raise Exception("CAPTCHA present on initial page load - unable to proceed")
```

### 2. Before Login
**Location:** `browser_controller.py:594-596`
```python
if await self._detect_captcha():
    if not await self._wait_for_captcha_resolution():
        raise Exception("CAPTCHA detected before login - unable to proceed")
```

### 3. After Login Click
**Location:** `browser_controller.py:610-613`
```python
if await self._detect_captcha():
    if not await self._wait_for_captcha_resolution():
        raise Exception("CAPTCHA during login - unable to proceed")
```

### 4. After Email Submission
**Location:** `browser_controller.py:622-625`
```python
if await self._detect_captcha():
    if not await self._wait_for_captcha_resolution():
        raise Exception("CAPTCHA after email - unable to proceed")
```

### 5. After Successful Login
**Location:** `browser_controller.py:635-638`
```python
if await self._detect_captcha():
    if not await self._wait_for_captcha_resolution():
        raise Exception("CAPTCHA after login - unable to proceed")
```

## Usage Notes

### For Developers

1. **Detection is Automatic**: No manual triggering needed, CAPTCHA detection runs at critical points
2. **Manual Resolution Only**: The system does NOT attempt to solve CAPTCHAs automatically
3. **Timeout is Generous**: Default 30 minutes allows plenty of time for manual solving
4. **Language Support**: Supports both English and Russian CAPTCHA text

### For Users

1. **Watch for Warnings**: Look for CAPTCHA warnings in console output
2. **Solve Manually**: Complete the CAPTCHA in the browser window
3. **System Continues**: After solving, automation resumes automatically
4. **Timeout Alert**: If you don't solve within 30 minutes, operation fails

## Error Handling

If CAPTCHA is not resolved in time:
```
❌ CAPTCHA resolution timeout after 1800s!
Exception: CAPTCHA after login - unable to proceed
```

**Recovery Options:**
1. Restart the operation
2. Increase timeout if needed
3. Check if CAPTCHA is triggering too frequently (may indicate IP/behavior issues)

## Best Practices

1. **CDP Mode**: Use Chrome DevTools Protocol mode to maintain browser session
2. **Session Persistence**: Login once, reuse session to minimize CAPTCHA triggers
3. **Reasonable Delays**: Don't automate too aggressively to avoid CAPTCHA triggers
4. **Monitor Logs**: Watch for CAPTCHA warnings to understand frequency

## Future Improvements

Potential enhancements (not currently implemented):

- [ ] Configurable timeout per CAPTCHA type
- [ ] Audio CAPTCHA support
- [ ] Notification system for CAPTCHA appearance
- [ ] Statistics on CAPTCHA frequency
- [ ] Integration with CAPTCHA solving services (optional)

## Related Files

- Implementation: `src/chatgpt_automation_mcp/browser_controller.py`
- Lines: 458-556 (detection and resolution)
- Integration: Lines 226-638 (login flow)

---

**Last Updated:** November 14, 2025
**Status:** ✅ Implemented and Active
