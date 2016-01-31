# Transplant Management

## What

To facilitate the transplant of renal organs to patients with acute renal failure. This domain is extracted from a larger application managing renal patient care.

## Scope

Transplant management begins with an assessment of the transplant recipient and ends with the follow-ups for the transplant operation.

The scope of the example handles the case of a living donation only, and excludes cadaveric and altruistic donations.

## Steps

Pre-condition(s):

- A patient has been recorded in the Patient Management module.

Steps:

- A `Recipient` has an `Assessment` to determine if they are medically applicable for a transplant.
- A `Recipient` is registered on the `Wait List` and the `Registration Status` is set to active.
- A potential `Donation` is recorded for the `Recipient`. The `Donation` requires the role of a  `Living Donor` played by another `Patient`. A `Recipient` may have many pending `Donations`.
- A `Living Donor` has an `Assessment` to determine if they are medically applicable (i.e. a "match") to make a `Donation` to the `Recipient`.
- The `Donation` is approved if the `Assessment` determines the `Donor` is medically applicable.
- The approved `Donation` results in two operations to be scheduled with a `Transplant Provider`, one each for the `Recipient` and the `Living Donor`.
- After the `Operation` is completed, both the `Recipient` and the `Living Donor` will receive
multiple `Operation Follow-up`s with an `After Care Provider`.

## Events

- Recipient Assessment
- Wait List Registration
- Donation
- Donor Assessment
- Recipient Operation
- Donor Operation
- Recipient Operation Follow-up
- Donor Operation Follow-up

## Guided Tour

- `Patient`: a patient is a person recorded in the Patient Management domain. They have unique ID to identify them as a patient, within the hospital (e.g. hospital_number) and within the national health system (e.g. national_health_number).
- `Recipient`: is a patient with acute renal failure requiring a transplant.
- `Recipient Assessment` (aka workup): is required to determine if recipient is medically applicable for a transplant. A successful assessment is required to be registered on the wait list. The assessment is used in the future to match potential donors.
- `Registration`: adds the recipient to the queue of other recipients waiting for a transplant.
- `Registration Status`: records the status history of the registration, e.g. pending, operation_scheduled, completed.
- `Wait List`: is the queue of transplant recipients.
- `Donation`: links a recipient with another patient who is a potential living donor.
- `Living Donor`: a patient who is potential or completed a donation of a renal organ to another patient, the recipient.
- `Donor Assessment`: determines if the living donor is medically applicable to donate a renal
organ to the recipient. This is commonly known as a "match" which infers a match on such physiological parameters as blood type. However, the assessment also determines if the potential living donor is able to survive an operation and live in good health after the operation.
- `Recipient Operation`: the recipient's operation to remove the failing renal organ and insert a donated renal org.
- `Donor Operation`: the operation to remove the working renal organ.

- `Tranplant Provider`: the site in which an operation for the recipient or donor will take place.
- `After Care Provider`: the site in which the recipient or donor will receive a follow-up (post-operation consultation).
- `Site`: a hospital or clinic.

## Feature List

- Record the registration for a patient on the waitlist
- Change the status of a patient's registration
- Determine the pending registrations on the waitlist
- Determine the total pending registrations on the waitlist
- Start an assessment for a recipient
- Complete the verification of recipient's assessment
- Record the donation for a living donor
- Assign recipient to the donation
- Start an assessment for a living donor
- Complete the verification of recipient's assessment
- Determine the possible donors for a recipient
- Schedule an operation for the recipient
- Schedule an operation for the donor
- Record the follow-up for the recipient's operation
- Record the follow-up for the living donor's operation
