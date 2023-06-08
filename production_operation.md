# Production operation

In this explainer, we’ll review the features available in the Protected Audience services and define the key components for production readiness, so that these services are operable at the desired reliability rate.

Our goal is to build a set of mitigations and plans so that when outages happen, they can be gracefully and quickly addressed, rather than trying to prevent outages or build “perfect” servers outright.


## Design principles

These pillars are presented in alphabetical order, rather than order of importance.


### Change management

_We’ll offer management tools to address service outages, which typically occur when there’s a system change._



*   Permissions and roles: to control who can make changes and when.
*   Server release process: this covers how a new release candidate is created and how the candidate is tested to certify that it’s ready and has no regressions.  Also, the ability to run "canary" jobs to test out a new release at a small scale before it goes to full production and understand how rollbacks will work.
*   Lifecycle management: releases will not be supported forever, and so we need to work out what commitments there will be and how old releases will be deprecated. This includes the handling of breaking versus non-breaking changes.


### Developer experience

_We’ll create a positive experience for developers by responding to open issues and offering a well-built service with the ability to contribute code._



*   Repository structure: the open source repositories for the servers should be consistent and integrated with tooling. For example, for code indexing and search, continuous integration, continuous build, etc.
*   Community contribution policies: these should be consistent between servers, especially where they share shared repositories for common components.
*   Debugging tools: these are especially tricky to build for Protected Audience services because of the deliberate limitations of what the servers expose about their running state. We plan to publish separate explainers (e.g. [Debugging protected audience API sevices][1]) on this topic.


### Incident management

_We’ll offer playbooks for service operators and define emergency procedures_



*   Support structures: when there is an outage, how can server operators report it for triage and remediation?
*   Playbooks: for server operators there should be a well defined set of common alerts and how/when to mitigate them.
*   Pre-planned mitigations: a set of responses that are expected to be useful.  For example, to roll back to a previous server version or to restore corrupt data.


### Reliability

_We’ll create a system that meets reliability requirements._



*   Reliability ranges: the server operators are responsible for deciding the reliability targets that they’re aiming for, e.g. uptime. The servers should be able to meet a range of different reliabilities; this is important because often extremely high reliability comes with significant costs.
*   Monitoring and alerting: the ability to detect when servers fail, or stray from reliability targets, is necessary. Without this server operators can’t tell if the servers are performing correctly. See [Monitoring protected audience API sevices][2] for details.
*   Risk management: past outages will be tracked and documented together with changes that come out of them. This both to track system health over time and to make sure that known bugs are corrected.
*   Graceful degradation: when servers are overwhelmed they should be able to shed traffic in a predictable way (while also alerting operators that they’re in this state, as above).
*   Scale testing: it’s useful to know where the limits of the servers are. For example, how much traffic can one single instance handle before it’s overwhelmed?


### Resources

_We’ll provide information as to what resources the server will use and how best to set it up to use them efficiently._



*   Sensible default deployment configurations: to make sure that common mistakes are avoided.
*   Ongoing benchmarking: to prevent performance regressions.
*   Cost prediction: to estimate the Cloud costs required to run certain workloads.


[1]: https://github.com/privacysandbox/fledge-docs/blob/main/debugging_protected_audience_api_services.md
[2]: https://github.com/privacysandbox/fledge-docs/blob/main/monitoring_protected_audience_api_services.md
