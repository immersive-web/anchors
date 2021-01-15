# Security and Privacy Questionnaire

This document answers the [W3C Security and Privacy
Questionnaire](https://www.w3.org/TR/security-privacy-questionnaire/) for the
WebXR Anchors Module specification.

**What information might this feature expose to Web sites or other parties, and for what purposes is that exposure necessary?**

In Augmented Reality frameworks, the understanding of the user's environment may be constantly evolving as the framework obtains more data. This API allows web applications to inform the user agents that specific points in users' environment should be tracked, and as the understanding changes, it allows the applications to query the latest location of tracked points relative to a reference space. For details, see [explainer](https://github.com/immersive-web/anchors/blob/master/explainer.md) and the [specification](https://immersive-web.github.io/anchors/).

Free-floating anchors should not directly expose any information about the user's environment. In case of anchors attached to hit test results, this feature may be used to indirectly expose information about user environment, similar to [hit-test API](https://github.com/immersive-web/hit-test/).

**Is this specification exposing the minimum amount of information necessary to power the feature?**

Yes. There isn't a lot of mitigations that can be done. The API allows creation of free-floating anchors and anchors attached to hit test results. Creation of anchors attached to hit test results relies on mitigations already provided by the [hit-test API](https://github.com/immersive-web/hit-test/) - no new mitigations are introduced by anchors API.

**How does this specification deal with personal information or personally-identifiable information or information derived thereof?**

There are no direct PII exposed by this specification. The mapping of the user's environment is the only derived information that could be done via this API, assuming that the application leverages anchors attached to hit test results.

**How does this specification deal with sensitive information?**

This specification does not directly expose sensitive information to the web applications.

This feature of WebXR has to be listed when requesting an XR session which
allows the user agent to show a prompt specifically requesting user approval.

**Does this specification introduce new state for an origin that persists across browsing sessions?**

No.

**What information from the underlying platform, e.g. configuration data, is exposed by this specification to an origin?**

None.

**Does this specification allow an origin access to sensors on a user’s device**

No. However, in order to provide a good immersive experience using AR features, the user agent may need to use sensors. The origin has never a direct access to them as part of this specification.

**What data does this specification expose to an origin? Please also document what data is identical to data exposed by other features, in the  same or different contexts.**

This specification isn't directly exposing any data to the origin but can be
used to get information about the user's physical environment.

**Does this specification enable new script execution/loading mechanisms?**

No.

**Does this specification allow an origin to access other devices?**

No.

**Does this specification allow an origin some measure of control over a user agent’s native UI?**

No.

**What temporary identifiers might this this specification create or expose to the web?**

None.

**How does this specification distinguish between behavior in first-party and third-party contexts?**

It is an extension to WebXR which is by default blocked for third-party contexts.

**How does this specification work in the context of a user agent’s Private Browsing or "incognito" mode?**

The specification does not mandate a different behaviour.

**Does this specification have a "Security Considerations" and "Privacy Considerations" section?**

Yes.

**Does this specification allow downgrading default security characteristics?**

No.

**What should this questionnaire have asked?**

N/A.
