# Rivet RFCs
> **NOTE**: The Rivet team is in the early stages of implementing this RFC process. Feedback is very welcome.

This is a place to discuss major changes to Rivet. By "major" we mean significant changes either to public APIs or internal implementation details, particularly that involve breaking changes.

Most changes that can be implemented in a backward-compatible way don't need to go through the RFC process and can rely on [issues](https://github.com/indiana-university/rivet-source/issues) and [pull requests](https://github.com/indiana-university/rivet-source/wiki/Pull-requests).

The RFC process something we're experimenting with as we start development on the next major version of Rivet. The core Rivet team will primarily be using this process as a way to sketch out and plan a handful of major updates to the Rivet `2.x.x` codebase. **It is highly subject to change**.

Our RFC process is based on the RFC process adopted by many prominent open-source projects like [Svelte](https://github.com/sveltejs/rfcs), [React](https://github.com/reactjs/rfcs), [Ember](https://github.com/emberjs/rfcs), and others.

## The process
1. Copy `0000-template.md` to `0000-my-new-feature.md` where `my-new-feature` is a short descriptive name that describes the new feature or change.
1. Fill in the RFC template making sure to add as much detail as possible, especially to the _Detailed design_ section which should be the bulk of the RFC's content.
1. Submit a pull request and ask for design feedback.
1. Build a consensus around your RFC and integrate the feedback you receive.
1. Once your RFC has been reviewed by the team, they will decide on whether it is a candidate for inclusion in Rivet. RFCs that become candidates will enter a _Final comment period_ lasting 5 business days signaled by a comment on the open pull request, a `final comments` label, and an announcement to the Rivet email list.
1. Any substantial changes that might result from comments or feedback during this time may trigger an extension of the _Final comment period_. After the _Final comment period_ is closed an RFC may be rejected by the team based on any new insights that come out of the comment period. If the RFC is rejected, someone from the team will respond in a comment summarizing the rationale behind their decision.
1. Once accepted, a team member will merge the RFCs associated pull request, at which point the RFC will become "active". "Acitive" means that the Rivet team will implement the RFC and will prioritize the development according to their current workload.

As a pull request the RFC will receive design feedback from the larger community, and the author should be prepared to revise it in response.
