# Code replay attack on the myGovID Scheme

Ben Frengley (Masters student, University of Melbourne)   

Vanessa Teague (CEO, Thinking Cybersecurity Pty Ltd and the A/Prof (Adj.) Australian National University)

## Summary

We explain a replay attack on the Australian Tax Office's myGovID scheme.  When a user tries to use the myGovID scheme to log in to a website under the attacker's control, the attacker can immediately log in as the user via myGovID at any other site.  The attack relies on the malicious site's ability to replay the 4-digit code that the myGovID scheme displays.

Although the attack is visible to a vigilant user who knows the protocol, we believe that most ordinary users' logins would be successfully hijacked.  At the server side, the login would be indistinguishable from a legitimate login from the user, so the attack is impossible to detect (excluding surveillance-based detection by device fingerprinting, login location, etc).

This [simple video](https://www.youtube.com/channel/UCGu4OlaZEVAWyjRV3FzCbDA) shows nontechnical users how to protect themselves. 

## Attack scenario
Suppose Alice wants to log in to nottrustworthy.com, using myGovID.  In the language of the Trusted Digital Identity Framework, nottrustworthy.com is the relying party (RP), Alice is the user, the ATO provides the Identity Exchange (IdX), and myGovID is the (sole) Identity Provider (IP).  The myGovID system uses a client app that Alice runs on her phone. nottrustworthy.com does not need to be an authentic RP integrated with myGovID; instead, it only needs to appear to Alice as if she can log into it using myGovID.

The adversary controls nottrustworthy.com and wishes to log in fraudulently, as Alice, at some other site, which we will call AlicesTaxService.gov.au.  Assume AlicesTaxService.gov.au is an authentic RP in the myGovID system, such that users can use myGovID to log in.

We assume that Alice already has the myGovID app installed and is somewhat familiar with its use but not an expert in its trust assumptions.

## Attack details

The adversary edits the web page at nottrustworthy.com to present a fake button inviting users to log in with myGovID.  (It is easy to copy a button that perfectly resembles the real one.)  Instead of honestly redirecting users to mygovid.gov.au, the adversary makes up a frame or page on their own website that resembles a myGovID login and asks for the user's email.  Again, this could perfectly copy the real myGovID site and say "Login with your myGovID to continue."  **A diligent user who knows this should come from mygovid.gov.au can detect this, but unless Alice knows exactly how the protocol works there is nothing suspicious about an email address request from a website she intended to interact with.**

The attack proceeds as follows.

1. When Alice enters her email, the attacker (either by hand or in an automated way) goes to AlicesTaxService.gov.au, clicks on 'Log in with myGovID,' waits for the (honest) redirect to mygovid.gov.au, and enters Alice's email address.

2. The myGovID system displays a 4-digit code, intended for Alice, on the mygovid.gov.au page that the adversary is reading.

3. The attacker reads the code and replays it to Alice, on the page at nottrustworthy.com that Alice is looking at, in a way that makes it appear to be a legitimate code from myGovID.

4. Alice reads the code and enters it into her app when requested.

5. The attacker will now be logged in to AlicesTaxService.gov.au as Alice.

6. In order to hide the attack completely from Alice, the attacker could show Alice a successful login at nottrustworthy.com.

The crucial design flaw is that when Alice's myGovID app receives an authorisation request and invites Alice to enter her 4-digit code, there is nothing in the app's user interface that tells her the name of the entity (RP) seeking authorisation. Alice thinks that she is consenting to log in to nottrustworthy.com. However, the myGovID system (both the IdX and the IP) are conveying the attacker's authorisation request from AlicesTaxService.gov.au.

## Analysis of impact
This attack is detectable by a diligent user who understands the protocol well enough to know that they should only accept 4-digit codes from mygovid.gov.au.  However we believe that there are very few users in this category, because it is a counter-intuitive protocol designed to reverse the information flow relative to what users are accustomed to.

Users are generally told (from primary school) always to check carefully that they are visiting the right website when they are about to enter their login credentials.  In practice maybe they do not always do this well, and most people don't know how to check for TLS, but browsers are getting better at this - for example, Firefox and Chrome both now warn when the user visits a not-TLS-protected site, or when a login and password is solicited in a way that seems suspicious.  Common email clients warn when a link does not go where it looks like it goes.  So most browsers and email clients put reasonable effort into thwarting the most obvious attacks on the traditional password-based information flow.  This is imperfect but at least most educated people (including primary school children) are somewhat aware of the problem.

The myGovID system aims to alleviate this problem (we assume) by reversing the information flow, so users never enter their password or 4-digit code into anything except their app.  This is a noble goal, but the implementation introduces another equivalent problem.

The main reason this is worse than the standard redirect-to-fake-login-site attack is that the information flow is so counter-intuitive and non-standard that users are much less likely to notice - we all know we are not supposed to enter credentials into websites we do not trust, but we have no intuition about whether we are supposed to enter a number from a website we semi-trust into an app we trust.  Also none of the browser-based defences against the redirect-to-fake-login attack would work against this attack.

There is nothing intuitively suspicious about getting a 4-digit code from a website you were trying to log in to, when that is standard in the typical authentication process when using myGovID. The user trusts the app, so the fact that they receive a notification from the app about the login may even alleviate their concern. A particularly knowledgeable user may notice that the code does not come from mygovid.gov.au, but otherwise there is nothing suspicious: neither the notification nor the code entry in the app provide any indication of which website the code applies to.

Even in normal circumstances, the myGovID protocol can be confusing to the user &mdash; starting an authentication process at an RP, abandoning it at code entry, and starting a new authentication process at the same RP (e.g., by getting to the code entry page then clicking the Cancel button, then entering the same email) results in an invalid code entry popup in the app, which when closed will immediately yield another, totally indistinguishable, code entry popup, which this time is valid. In that scenario both code entry popups are honest and correspond to authentic login requests at a registered and trusted RP. However, they are entirely indistinguishable: nothing indicates to the user which RP they are from, when the login was initiated or that the first code entry popup is no longer valid and that there is a second popup awaiting user attention. Entering the code from the second login attempt at the first code entry popup yields a cryptic "Something is wrong with the code. Try again," error message with no indication of what the error is and no reason for the user to expect an error to occur.

This kind of confusing user experience teaches even normally vigilant users to ignore things that might otherwise seem odd, and myGovID's lack of context for login requests exacerbates this issue, which makes this attack more concerning.


## Mitigations and their impact

### Short term - for users

Users are advised not to use the myGovID system until the protocol is patched.   

If use of the myGovID system is unavoidable, each user should check diligently that the 4-digit code they are about to enter comes from a TLS-protected URL at https://mygovid.gov.au.  This unlikely to work in practice for most users, who will struggle to recognise a secure website with the right URL.


### Short term - for government

Even if all users carefully perform the check above, a randomised version of the same attack could still be attempted: the malicious website faithfully (but with a small delay) passes the user on to the real mygovid.gov.au login site, while more quickly trying to log on as that user elsewhere.  Unless there are careful protections in place to ensure that the 4-digit codes are never the same, there is a chance of 1/10,000 that the codes will match, higher if we assume an opportunity for a few guesses.  Without having seen the code generation algorithm, we cannot tell whether such a mitigation is in place or not, but if not it should be added urgently.

The app should also be updated immediately with the following simple mitigation:

**When an authentication request is received, tell the user what website is requesting it.**

Technically, this is incompatible with the stated goals of the Trusted Digital Identify Framework, in which the Identity Exchange (provided by auth.ato.gov.au in our example) obscures the identity of the Relying Party (nottrustworthy.com in our example) from the Identity Provider (myGovID in our example).  However, the ATO's Identity Exchange leaks the RP's identity to myGovID via the HTTP Referer header, so this information is already available and can be used as a mitigation.  Hiding the RP's identity from the app seems to be a very low priority goal compared with preventing fraudulent logins.

Attempting to certify trustworthy RPs would not help unless users have an simple way of checking who has been certified that can be easily included in a typical authentication process.

### Long term

In the long run, the TDIF and all its current implementations should be deprecated and replaced with an open standard such as OpenID Connect or a protocol modelled on that of a nation with an existing secure public key infrastructure such as Belgium or Estonia.  The implementation and design documentation should be openly available to the Australian public to allow for the identification and responsible disclosure of other vulnerabilities.

We have no reason to believe that this is the only, or the worst, vulnerability in this system. Its complex nature and desire to hide information makes enforcing and validating correct, secure behaviour close to impossible.

## Responsible disclosure history

This problem was disclosed on 19th August 2020 to the Australian Signals Directorate, with an indicative expectation of a 90-day disclosure period.  ASD communicated it to the ATO.  At a meeting on 18th September 2020, ATO told us they did not intend to change the protocol, at which point we immediately informed them that we would make a warning to users public on Monday 21st September.

## Acknowledgements

Thanks to Rod Teague and Andrew Conway for their help.  Thanks also to Yaakov Smith for helpful review of this work.

## Usage and Contacts

You are welcome to quote or reuse this material as long as you credit the original source.

Email contact: bfrengley [at] student.unimelb.edu.au or vanessa [at] thinkingcybersecurity.com 

"Disclosure.md" [readonly] 88L, 11217C
