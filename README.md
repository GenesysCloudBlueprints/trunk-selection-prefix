# Trunk Selection Prefix

This is a blueprint is a solution for idea INB-I-1146 (https://genesyscloud.ideas.aha.io/ideas/INB-I-1146). Genesys Cloud trunk selection is handled by configuration on Sites. Number Plans and Outbound Routes ultimately determine the target trunk by matching the dialed number against a prioritized Number Plan, classifying it, and attempting the highest priority trunk matching the classification.

Telephony Administrators can steer different types of calls through different trunks, assuming there is something unique to the dialed number scheme that can be matched against using one of the supported match types. Match types include Digit Length, E.164 Number List, Inter-Country, Intra-Country, Number List, or RegEx.

The following illustration shows the end-to-end user experience that this solution enables.

