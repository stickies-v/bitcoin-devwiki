The IRC meeting is every Thursday at 14:00 UTC (with some exceptions for skipping due to CoreDev).

The following is the general script used for running the meeting, text in `code formatting` can be pasted in directly.

* Start the meeting: `#startmeeting`. You should get a private message from `core-meetingbot`.
* Ping everyone:
    ```
    #bitcoin-core-dev Meeting: achow101 _aj_ amiti ariard aureleoules b10c BlueMatt brunoerg cfields darosior dergoegge dongcarl fanquake fjahr furszy gleb glozow hebasto instagibbs jamesob jarolrod jonatack josibake kallewoof kanzure kouloumos kvaciral laanwj LarryRuane lightlike luke-jr MacroFake Murch phantomcircuit pinheadmz promag provoostenator ryanofsky sdaftuar S3RK stickies-v sipa theStack TheCharlatan vasild
    ```
* Ask for last minute topics (usually there are none, replace `no` with the number when there are): `There are no pre-proposed meeting topics this week. Any last minute ones to add? Reminder that you can pre-propose a meeting topic by prefixing your message with #preproposedmeetingtopic`
* Priority projects are fixed topics. Do them one by one and ping the project's champion:
  * `#topic package relay updates (glozow)`
  * `#topic BIP 324 updates (sipa)`
  * `#topic libbitcoinkernel updates (TheCharlatan)`
  * `#topic assumeutxo updates (jamesob)`
* Ask about the ad-hoc high priority for review
  * `#topic Ad-hoc high priority for review`
  * `Anything to add or remove from https://github.com/orgs/bitcoin/projects/1/views/4`
* Around the time of a release, remind people about feature freeze and the release. Possibly replace ad-hoc priorities with a release priorities, e.g. `#topic 26.0 release priorities`
* When most recent topic appears to be finished, and there are no other topics, ask if there are any further topics: `Anything else to discuss?`
* End the meeting either when there are no topics left, or the time is 15:00 UTC: `#endmeeting`