# Dark Patterns & Manufactured Friction

Deceptive or manipulative UX harms the user even when the interface is "usable" and attractive. Flag it as a first-class finding, high priority regardless of surface polish. A flow that succeeds by tricking or wearing down the user has failed the user.

## The Test

> Does this design help the user's goal, or the business's goal at the user's expense?

Friction that protects the user (confirm before delete) is good. Friction that serves the business by obstructing the user (hard-to-find cancel) is a dark pattern. When the two goals conflict and the design quietly picks the business, that is the finding.

## Severity

Each pattern present is at least **P1**. Patterns that cause financial loss, unwanted commitments, or privacy harm — or that are outright deceptive — are **P0**. Report the pattern, the evidence, and a fix that realigns the flow with the user's intent.

## Patterns to Flag

| Pattern | What it looks like | Why it harms |
|---------|--------------------|--------------|
| **Roach motel** | Easy to get in (sign up, subscribe), hard to get out (cancel buried, requires a call/email) | Traps the user in unwanted commitments |
| **Forced continuity / hidden costs** | Silent trial→paid conversion; fees appearing only at the last step | Extracts money the user didn't knowingly agree to |
| **Sneak into basket** | Pre-added items, add-ons, or donations the user must remove | Charges for things not chosen |
| **Confirmshaming** | Decline options worded to guilt ("No, I don't want to save money") | Manipulates via shame, not information |
| **Preselected opt-ins** | Marketing/consent/data-sharing checkboxes checked by default | Manufactures consent the user didn't give |
| **Misdirection / false hierarchy** | The manipulative choice is the bright primary; the user's interest is a faint link | Steers away from the user's actual intent |
| **Trick wording / double negatives** | "Uncheck to not opt out" | Confuses the user into the wrong choice |
| **Nagging** | Repeated interruptions to enable notifications, rate, upgrade | Wears the user down until they relent |
| **Forced action / privacy zuckering** | Must grant broad permissions or share data to proceed unrelated task | Coerces disclosure irrelevant to the goal |
| **Fake urgency / scarcity** | Countdown timers, "only 2 left!" that aren't real | Pressures decisions with false information |
| **Fake social proof** | Invented activity ("12 people viewing"), fabricated reviews | Manufactures trust dishonestly |
| **Obstruction** | Extra steps inserted only to discourage a user-favorable action (cancel, unsubscribe, data export) | Punishes legitimate choices |
| **Disguised ads** | Ads styled as content or system UI / native controls | Tricks clicks the user didn't intend |
| **Hard-to-cancel / unsubscribe** | No in-product cancel; unsubscribe link broken or multi-step | Violates the user's right to leave |
| **Gatekept exit / interstitial walls** | Sign-up wall, cookie wall, or interstitial before the user can see value | Blocks the goal to force a commitment |

## Distinguishing Protective Friction from Dark Patterns

| Legitimate friction (keep) | Dark pattern (flag) |
|----------------------------|---------------------|
| Confirm before deleting data | Confirm dialog whose safe default is the destructive one |
| Requiring a password to change billing | Requiring a phone call to cancel |
| Clear, unchecked consent boxes | Pre-checked consent boxes |
| Honest "are you sure?" with a neutral decline | Decline button that guilt-trips or hides |
| Real low-stock indicator tied to inventory | Countdown/scarcity that resets or is fabricated |

## Reporting

Name the specific pattern, not "this feels manipulative."

**Good:**
> **[F-01] Cancel path is a roach motel** · Gap · Dimension: Dark Pattern · P0
> **Location:** Settings → Billing → Cancel
> **Evidence:** "Manage plan" offers Upgrade and Pause but no Cancel; canceling requires emailing support (found only in the help center).
> **Impact:** Users cannot end a paid commitment in-product; this traps them and erodes trust (and may breach click-to-cancel rules).
> **Recommendation:** Provide an in-product Cancel with parity to the sign-up flow — same number of steps, no forced call, immediate confirmation.
