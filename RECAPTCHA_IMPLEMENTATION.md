# reCAPTCHA v3 Implementation Guide

## Overview
reCAPTCHA v3 has been successfully implemented in your webpage for three critical actions:
1. **Register** - User registration with phone number
2. **Verify OTP** - OTP verification
3. **Submit Challenge** - Final challenge submission

## What Was Changed

### 1. Added reCAPTCHA Script (Line ~52)
```html
<!-- reCAPTCHA v3 -->
<script src="https://www.google.com/recaptcha/api.js?render=YOUR_SITE_KEY_HERE"></script>
```

### 2. Added reCAPTCHA Helper Function
A new async function `getRecaptchaToken(action)` was added to fetch tokens for specific actions:
- `register` - For user registration
- `verify_otp` - For OTP verification  
- `submit_challenge` - For challenge submission

### 3. Updated Three API Calls

#### Register API Call
- **Action**: `register`
- **Location**: Inside the "Submit" button click handler (sendOtpBtn)
- **Added field**: `recaptchaToken` in the request body

#### Verify OTP API Call
- **Action**: `verify_otp`
- **Location**: Inside the "Verify OTP" button click handler (verifyOtpBtn)
- **Added field**: `recaptchaToken` in the request body

#### Submit Challenge API Call
- **Action**: `submit_challenge`
- **Location**: Inside the "Continue" button click handler (continueBtn)
- **Added field**: `recaptchaToken` in the request body

## Important: Configuration Required

### Replace the Site Key
You **MUST** replace `YOUR_SITE_KEY_HERE` in TWO places:

1. **In the script tag** (around line 52):
```html
<script src="https://www.google.com/recaptcha/api.js?render=YOUR_ACTUAL_SITE_KEY"></script>
```

2. **In the JavaScript constant** (around line 905):
```javascript
const RECAPTCHA_SITE_KEY = 'YOUR_ACTUAL_SITE_KEY'; // Replace with your actual site key
```

### How to Get Your Site Key
1. Go to [Google reCAPTCHA Admin Console](https://www.google.com/recaptcha/admin)
2. Register your site if you haven't already
3. Select **reCAPTCHA v3**
4. Add your domain(s)
5. Copy the **Site Key** (public key)
6. Replace `YOUR_SITE_KEY_HERE` with your actual site key

## Backend Implementation Required

Your backend API must be updated to:

1. **Receive the reCAPTCHA token** in the request body:
   - Field name: `recaptchaToken`
   - Type: String

2. **Verify the token** with Google's API:
```javascript
// Example Node.js verification
const axios = require('axios');

async function verifyRecaptcha(token, action, expectedAction) {
    const secretKey = 'YOUR_SECRET_KEY'; // From reCAPTCHA admin
    const verificationURL = `https://www.google.com/recaptcha/api/siteverify?secret=${secretKey}&response=${token}`;
    
    const response = await axios.post(verificationURL);
    const { success, score, action: returnedAction } = response.data;
    
    // Verify success, score threshold (e.g., 0.5), and action matches
    if (success && score >= 0.5 && returnedAction === expectedAction) {
        return true;
    }
    return false;
}
```

3. **Handle verification results**:
   - Score >= 0.5: Likely human (allow action)
   - Score < 0.5: Likely bot (reject or flag)
   - Failed verification: Reject request

## CSRF Token - Unchanged

The existing CSRF token implementation remains **completely intact**:
- CSRF tokens are still fetched and sent with all requests
- The `secureApiCall()` function continues to work as before
- Both CSRF and reCAPTCHA work together for dual protection

## Testing

1. **Test with reCAPTCHA disabled** (before adding site key):
   - Forms will still work but tokens will be null
   - Console will show warnings about missing tokens

2. **Test after adding site key**:
   - Open browser DevTools → Console
   - Look for reCAPTCHA token logs
   - Verify tokens are being sent in Network tab

3. **Test backend verification**:
   - Ensure backend validates tokens properly
   - Check that legitimate users can submit successfully
   - Verify suspicious activity is blocked

## Security Benefits

1. **Bot Protection**: Prevents automated form submissions
2. **Score-based Risk Assessment**: reCAPTCHA v3 assigns risk scores (0.0 to 1.0)
3. **No User Interaction**: Works invisibly in the background
4. **Combined with CSRF**: Dual-layer protection against different attack vectors

## Troubleshooting

### reCAPTCHA not loading
- Check if the site key is correct
- Ensure domain is registered in reCAPTCHA admin
- Check browser console for errors

### Tokens showing as null
- Wait for reCAPTCHA to load (check `grecaptcha` object exists)
- Verify site key is correctly set in both places

### Backend rejecting requests
- Ensure backend is updated to accept `recaptchaToken` field
- Verify secret key is correct on backend
- Check score threshold isn't too strict

## Additional Resources

- [reCAPTCHA v3 Documentation](https://developers.google.com/recaptcha/docs/v3)
- [reCAPTCHA Admin Console](https://www.google.com/recaptcha/admin)
- [Score Interpretation Guide](https://developers.google.com/recaptcha/docs/v3#interpreting_the_score)

## Next Steps

1. ✅ Get your reCAPTCHA v3 site key and secret key
2. ✅ Replace `YOUR_SITE_KEY_HERE` in the code (2 places)
3. ⚠️ Update your backend to verify reCAPTCHA tokens
4. ⚠️ Test thoroughly in development environment
5. ⚠️ Deploy to production and monitor

---

**Note**: The CSRF token implementation remains unchanged and continues to provide protection against CSRF attacks. reCAPTCHA v3 adds an additional layer of bot protection.
