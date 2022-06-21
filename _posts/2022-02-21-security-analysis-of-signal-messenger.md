---
layout: post
---
In this article we'll be looking at the security of the instant messenger [Signal](https://signal.org/en/). For this I'll be referring to Signal's [third-party audits](https://community.signalusers.org/t/wiki-overview-of-third-party-security-audits/13243), research papers and articles concerning the protocol and applications.

## Contents
- [Messaging protocol](#messaging-protocol)
    - Extended triple Diffie-Hellman (X3DH) key agreement
    - Double ratchet algorithm
    - Conclusion
- [Other areas](#other-areas)
    - Contact discovery
    - Database protection
- [Closing thoughts](#closing-thoughts)


## [Messaging protocol](#messaging-protocol)

The [Signal protocol](https://github.com/signalapp/libsignal-protocol-c) is secure instant messaging protocol used by many applications including [WhatsApp](https://www.whatsapp.com/security/WhatsApp-Security-Whitepaper.pdf), [Skype](https://az705183.vo.msecnd.net/onlinesupportmedia/onlinesupport/media/skype/documents/skype-private-conversation-white-paper.pdf) and [Messages by Google](https://www.gstatic.com/messages/papers/messages_e2ee.pdf).  It is based on two underlying protocols, the [extended triple Diffie-Hellman (X3DH) key agreement](https://signal.org/docs/specifications/x3dh/x3dh.pdf) and the [Double Ratchet protocol](https://signal.org/docs/specifications/doubleratchet/doubleratchet.pdf). It's primary aim is to keep eavesdroppers with mallicious intent like [Mallory](https://crypto.stackexchange.com/questions/85875/there-is-alice-and-there-is-bob-but-what-is-the-name-of-the-malicious-user) from decrypting messages while in transit.

The first formal security audit was done by Cohn-Gordon et al. in 2016[^1] and states the following:
> [...] Our analysis proves that several standard security properties are satisfied by the protocol, and we have found no major flaws in its design, which is very encouraging.

However, a lot more research has been done since then on the underlying cryptographic protocols mentioned earlier. A few of those papers will be discussed in the following sections. 

### Extended triple Diffie-Hellman (X3DH) key agreement

The [Extended triple Diffie-Hellman (X3DH) key agreement](https://signal.org/docs/specifications/x3dh/x3dh.pdf), as the name states, is an extended form of an established encryption protocol, called the Diffie-Hellman (DH) protocol. It creates a shared secret key between two parties who then mutually authenticate each other based on their public keys. Writing this one line doesn't do justice to it's elegance, so here are [two](https://youtu.be/NmM9HA2MQGI) [videos](https://youtu.be/Yjrfm_oRO0w) by Dr Mike Pound from Numberphile which explain DH and here's a [Medium article](https://medium.com/asecuritysite-when-bob-met-alice/alice-whispers-to-bob-at-the-core-of-privacy-in-the-21st-century-is-extended-triple-51736dad527d) by Prof Bill Buchanan OBE explaining X3DH and highlights the nuances between it and simple DH. 

A Bachelor thesis by van der Have (2021)[^2] claims to establish some security goals of the X3DH protocol:
> In this thesis, we have considered X3DH in the Signal protocol, and derived a proof of security. From the proof it follows that X3DH provides secrecy and authentication.

However, one limitation to the above paper is that the author only proves [DH](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange)'s integrity against classical (realistic in the current timeframe) threats, which is well researched. Another is that they were unable to test out-of-order decryption, meaning [forward secrecy](https://en.wikipedia.org/wiki/Forward_secrecy) might not be preserved during the interval when the message has been received but has not been decrypted due to say, lack of network availability. Forward secrecy, very briefly, means that even if the private keys are compromised, the session keys won't be and the channel created to communicate will remain intact. 

Currently, in order to break DH, Mallory needs to solve a [discrete log problem](https://math.dartmouth.edu/~carlp/dltalk09.pdf). This employs extreme computation with expenses on the order of millions of dollars, something that is only feasible for government agencies[^3]. Even then, the benefits of this are limited, as forward secrecy ensures that keys are generated with every new message sent. Thus, there won't be a sustained compromise in the communication process, and Mallory would only a single message.

However, in a [post-quantum](https://csrc.nist.gov/projects/post-quantum-cryptography) scenario, X3DH, in its current form, would be breached. This is the subject of a research paper by Hashimoto et al. (2021)[^4] where the authors aim to improve forward secrecy, [deniability](https://en.wikipedia.org/wiki/Deniable_encryption)[^5] and establish post-quantum integrity. They summarise as follows:

> [...] This results in the first post-quantum secure replacement of the X3DH protocol.

Although this research wasn't able to improve upon forward secrecy much, but the other areas received signifiant leaps in security. 

Such studies could improve Signal's existing security massively, and we'll see some more examples of post-quantum research being done on the protocol in the following section.

### Double ratchet algorithm

The [Double ratchet algorithm](https://signal.org/docs/specifications/doubleratchet/doubleratchet.pdf) is used by two parties to exchange encrypted messages based on a shared secret key. The parties will use some key agreement protocol (such as [X3DH](https://signal.org/docs/specifications/x3dh/x3dh.pdf)) to agree on the shared secret key. Following this, the Double Ratchet will be used to send and receive encrypted messages. Like X3DH, this explainer does not suffice, so I leave Dr Mike Pound's [video](https://youtu.be/9sO2qdTci-s) again. 

A paper presented at EIConRusNW by [IEEE](https://www.ieee.org/) (2020) by Bobrysheva and Zapechnikov[^6] concludes the following:

> [...] The Double Ratchet algorithm is an elegant and straightforward protocol for providing classical security in messaging systems, although we can’t use it in post-quantum systems. 

In another paper, presented at [EUROCRYPT 2019](https://eurocrypt.iacr.org/2019/), the researchers Alwen et al.[^7] state that Singal's elegance comes from the fact that Forward secrecy and Post-compromise security (PCS) are not only achieved together, but also without sacrificing immediate decryption and message-loss resilience (MLR). Although Forward secrecy was looked into before, a few other terms require an explainer at this stage. They are as follows:

1. **Post-compromise security (PCS):** Once the communication channel, which was leaking secrets so to speak, has stopped doing so, the security is restored.
2. **Immediate decryption:** As it sounds, immediate decryption aims to decrypt messages upon being delivered with no-delay.
3. **Message-loss resilience (MLR):** The communication channel should be functional and secure even if a few messages were lost in transit.

This paper went further, quite like the paper by Hashimoto et al. mentioned above, to create a generalized double ratchet protocol, which ensures post-quantum integrity. In the introduction, the authors say the following:

>  As a result, in addition to instantiating our framework in a way resulting in the existing, widely-used Diffie-Hellman based Signal protocol, we can easily get post-quantum security and not rely on random oracles in the analysis.

### Conclusion

If you've read this article thus far, you'd have seen that the Signal protocol fends off the current classical threats effectively. And seeing as quantum computers won't become mainstream in at least the next decade (or two), I think Signal is safe. And papers cited above have highlighted how the protocol can be strengthened to preserve it's integrity in a post quantum world. 


## [Other areas](#other-areas)

### Contact discovery

We have seen that the Signal protocol keeps you safe from eavesdroppers. But Signal's contact discovery has serious privacy concerns which might lead to widespread user tracking. This was demonstrated in the 2021 research paper by Hagen et al. [^8] who write:

> Our script for Signal uses 100 accounts over 25 days
to check all 505 million mobile phone numbers in the US.
Our results show that Signal currently has 2.5 million users
registered in the US, of which 82.3 % have set an encrypted
user name, and 47.8 % use an encrypted profile picture.

Thus, using their script, the authors managed to find every Signal user registered in the US less than a month. However, as much as this result looks alarming, Signal fairs much better at protecting user privacy from contact discovery than alternatives like WhatsApp and Telegram. Unlike WhatsApp and Telegram who store contact information on their servers, Signal chooses not to. Also Signal only gives away a user's phone number while other platforms may give away profile pictures, about information and other additional data. 

This paper has not only highlighted the concerns, it has improved upon the methodology of contact discovery, which makes such data gathering much more difficult and not feasible for a threat actor. The authors also reached out to Signal who acknowledged that this was an issue inherent to contact discovery itself, but they made necessary changes which would make the methodology used by the authors a lot more time consuming. They also set-up "further defenses". 

### Database protection

A 2019 paper by Kaczyński[^9] highlights a different issue. One where a malicious actor has access to your device physicially. In such a situation, the conclusion is rather glaring:

> The research conducted and presented in this paper indicates the presence of
many threats to the Signal users who currently do not have any mechanisms to
protect the stored data, except those offered by the operating systems. This
situation allows not only for reconstructing the history of conversations but also
for impersonating one of the users participating in the communication using the
application. Such an attack will not be detected by the other side, because common
identifiers will not change.

Delving deeper, I found this related to a recent saga between Signal's developer, Moxie Marlinspike and Cellbrite, as mentioned in the paper, a tool which the malicious actor would use to extract data in this case. [This blog post](https://cyberlaw.stanford.edu/blog/2021/05/i-have-lot-say-about-signal%E2%80%99s-cellebrite-hack) by Riana Pfefferkorn of Stanford details all that unfolded. The way Signal claims to throw off Celbrite is by injecting a potential exploit or noise in every local Signal databse. This is all based on an assumption from [Marlinspike's reveal of the matter](https://signal.org/blog/cellebrite-vulnerabilities/) as Pfefferkorn states. Cellbrite has apparently pushed an [update](https://www.vice.com/en/article/qj8pjm/cellebrite-pushes-update-after-signal-owner-hacks-device) to mitigate this sort of a 'reverse-hack'.


I reached out to the author of the earlier paper for comment if these vulnerabilities still existed, and his answer was 'to my best knowledge - they are still there'. 

Another paper by Son et al. (2022)[^10], provides a proof of work based on the aforementioned concept. The authors demonstrate how data can be scraped using a decryption script which they published [here](https://github.com/hunjison/Signal-Extract-AndroidKeyStore). They state the following:

> As a result, we could decrypt all encrypted database, multimedia, log, and preferences files of Signal, Wickr, and Threema.

## [Closing thoughts](#closing-thoughts)

Thus, upon delving into Signal's security, I came away with the following: 

1. It has got the best privacy practices of any mainstream instant messenger. 

2. Signal protocol is one of the most secure instant messaging security protocols currently available. And with certain changes, it can be made post-quantum secure. Multiple messages in transit cannot be interepted currently, even by governmental agencies, and likely never will be feasible in the future.

3. Despite the flaws mentioned in [Other areas](#other-areas), having physical access to your device and a malicious actor trying to extract data from a sole application is beyond what most of our [threat models](https://thenewoil.org/threatmodel.html) call for. 

4. Signal has been making parts of their source code private lately, or pushing updates late[^11] [^12] as highlighted by Kaczyński in our conversation, which is not a good look. The justificiation provided by the developers are upto the end user to accept. Since, this article wasn't meant to be a discussion on open-source, I will refrain from making a comment on this. However, alternative apps like [Molly](https://molly.im/) aim to improve upon Signal's shortcomings while also allowing you to contact all Signal users. 

5. For casual communication, it is still the best instant messenger to have. 

<br>
<br>

---

#### Footnotes

{: data-content="footnotes"}

<!-- [^]: []() -->
[^1]: [A Formal Security Analysis of the Signal Messaging Protocol - Cohn-Gordon et al. (2016)](https://eprint.iacr.org/2016/1013.pdf)
[^2]: [The X3DH Protocol: A Proof of Security -  van der Have (2021)](https://www.cs.ru.nl/bachelors-theses/2021/Ferran_van_der_Have___4104145___The_X3DH_Protocol_-_A_Proof_of_Security.pdf)
[^3]: [Imperfect Forward Secrecy: How Diffie-Hellman Fails in Practice - Adrian et al. (2015)](https://weakdh.org/imperfect-forward-secrecy-ccs15.pdf)
[^4]: [An Efficient and Generic Construction for Signal’s Handshake (X3DH):Post-Quantum, State Leakage Secure, and Deniable - Hashimoto et al. (2021)](https://eprint.iacr.org/2021/616.pdf)
[^5]: [Further reading: On the Cryptographic Deniability of the Signal Protocol - Vatandas et al. (2020)](https://ieeexplore.ieee.org/document/7467371)
[^6]: [Post-Quantum Security of Messaging Protocols: Analysis of Double Ratcheting Algorithm - Bobrysheva and Zapechnikov (2020)](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=9039075)
[^7]: [The Double Ratchet: Security Notions, Proofs, and Modularization for the Signal Protocol - Alwen et al. (2019)](https://link.springer.com/chapter/10.1007/978-3-030-17653-2_5)
[^8]: [All the Numbers are US: Large-scale Abuse of Contact Discovery in Mobile Messengers - Hagen et al. (2021)](https://www.ndss-symposium.org/wp-content/uploads/ndss2021_1C-3_23159_paper.pdf)
[^9]: [SECURITY ANALYSIS OF SIGNAL ANDROID DATABASE PROTECTION MECHANISMS - Kamil Kaczyński (2019)](https://ijits-bg.com/contents/IJITS-No4-2019/2019-N4-06.pdf)
[^10]: [Forensic analysis of instant messengers: Decrypt Signal, Wickr, and Threema - Son et al. (2022)](https://www.sciencedirect.com/science/article/pii/S2666281722000166#fn1)
[^11]: [Signal-Server code not public since April 22 2020](https://github.com/signalapp/Signal-Android/issues/11101)
[^12]: [Improving first impressions on Signal](https://signal.org/blog/keeping-spam-off-signal/)
