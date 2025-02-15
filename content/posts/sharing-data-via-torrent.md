+++
date = '2025-02-14'
draft = false
title = 'Sharing Big Data via Torrent Protocol'
+++
## Introduction

My day job involves working with large datasets, and I often need to share terabytes of data with coworkers. That’s all well and good—provided your company doesn’t mind paying egress fees, sharing network folders, or swapping external hard drives. A few months ago, I needed to adapt some existing code for our use case. Before I could do that, I had to get it running with the original dataset which was just over a terabyte  in size.

This is where I found the inspiration for this article! The code, which was part of a graduate thesis for a student at Oregon State University, linked to a page with instructions to download datasets larger than 5GB via the BitTorrent Protocol.

>Datasets larger than 5GB deposited in ScholarsArchive@OSU will be distributed using the BitTorrent protocol. BitTorrent, in essence, splits the file into smaller pieces, distributes the smaller pieces to you, and then reconstructs them to recreate the whole - all in a way that reduces impact on our computer networks. In addition, each “piece” of the file carries a marker that helps verify the validity and integrity of the final file.

I thought this was a super interesting way to share large datasets in an academic setting so I decided to do some more digging. It turns out there is existing infrastructure to share academic data via torrents, notably <a href="https://academictorrents.com/">academictorrents.com</a>, which hosts over 127TB of research data available for download via the BitTorrent protocol. This platform provides a centralized location for researchers and institutions to share their work, making large datasets more accessible to the academic community.

While it might be a bit more challenging to find universities that openly advertise torrenting as a method for sharing data—like Oregon State University does—it’s clear that there is an increasing demand for decentralized file-sharing solutions in academia.

## Why use the Torrent Protocol to Share Data?

<details>
  <summary>Relevant xkcd ⬇️</summary>
  <img src="https://imgs.xkcd.com/comics/file_transfer.png" alt="xkcd File Transfer">
</details>

As mentioned by OSU above, using a torrent client breaks the files into small pieces and then reassembles them on the users end, but how does this reduce impact on networks?

To understand how this may impact a network, we first need to discuss how torrenting works in a bit more detail. <a href="https://medium.com/@edouard.courty/how-does-the-torrent-protocol-work-4ff40615d2ba" target="_blank">This Medium article</a> by Edouard Courty offers a great explanation - in essence when you download a file via BitTorrent, you’re not getting it from a single server but from multiple sources, often referred to as __seeds__ and __peers__. When you initiate a download, your Torrent client will reach out to a __tracker__, which helps it find other users who have the same Torrent file downloaded and are currently sharing it (seeds) or are currently downloading the same content (peers).

<img src="https://www.rapidseedbox.com/wp-content/uploads/how-torrenting-works-1.jpg" 
     alt="Peers and seeders diagram" 
     style="height: 40vh; width: auto;">

 If this is a bit confusing, take a look at this <a href="https://www.reddit.com/r/explainlikeimfive/comments/10bl0sb/eli5_how_do_torrents_work/" target="_blank">reddit post</a> by u/Dekkars in the r/ELI5 subreddit.

>You'd like to buy a book. One option is to go to the bookstore and spend money on it (direct download) but you don't want to do that.
>
>So you talk to your friend Alberto. Alberto knows everyone, and has a long list of all the people who own this book and are willing to photocopy a page (seeders).
>
>You send a letter to each person, asking for a specific page number (leeching).
>
>They send a letter back with that specific page. Sometimes you are missing a page, or it's unreadable. That's ok, you have that list and you can just ask someone else!
>
>Now you have all the pages of the book. Because everyone helped you, you now tell Alberto that you're willing to help. You give him your address, and you start getting letters asking for pages (becoming a seeder).
>
>That's fine. You can enjoy your book and help others get pages too.

This comes with some advantages: 

First, with a torrent, once the first few people start downloading, they also become seeders, meaning later downloaders can get pieces of the file from multiple sources instead of relying solely on your network. This has potential to vastly reduce the outgoing bandwidth over direct downloading, especially if multiple people are trying to download the data simultaneously.

Second, it can increase the reliability of downloads. If the data server goes offline, you can still collect file parts from seeders and peers, whereas with a direct download, a server crash may causing the download to fail. Speaking of not relying on one server, another advantage of decentralization is sharing the load among all users participating in the torrent. This can reduce costs and help reduce the effects felt by server limitations.

Finally, torrenting scales exceptionally well. If a large number of users try to download data from a central server at the same time, this can put immense load on a network, causing performance degradation. However, when downloading data via BitTorrent, each client is sharing bandwidth, reducing the load on any one server.

Amazing! We have:

1. High performance
2. High reliability
3. High availability
4. High scalability
5. Lower network load

## Why Would We Ever _Not_ Use the Torrent Protocol to Share Our Data?

They say sharing is caring, and when it comes to torrenting, that couldn't be more true. If you are the only seeder and there are no other peers, then the downloader is using only your bandwidth to retrieve the file. This scenario is essentially the same as a centralized, direct download because you're the sole source of the data. That means having multiple seeders and peers are essential for BitTorrent to work effectively. For professionals and hobbyists to effectively share data via torrents, the data must be in demand. If you work in a highly specialized field with few potential downloaders or handle proprietary company data, maintaining a healthy number of seeders and peers can be a challenge. So, sharing your data via the BitTorrent protocol might seem ideal in theory, but real-world results can vary.

When downloading chunks of data from different peers there is always a chance that one of those chunks may be faulty. Thankfully, the risk of this is minimal as the BitTorrent protocol has built-in hash checks to verify each piece of the file, preventing most corruption issues.

Malware and fake torrents can also be an issue, you should always verify that you are receiving your data from a trusted source and validate your `.torrent` by verifying the source and inspecting it for "red flag" extensions, in particular, executables such as `.exe`, `.bat`, `.scr`, `.js`, `.vbs`. A great security measure is to use a remote server, known as a seedbox, to download and upload torrents quickly and safely. If you choose to use your local machine it's important to remember that your IP address is visible to other peers, make sure to always use a VPN to help anonymize your connection and minimize the risk of exposure to malicious actors.

## Final Thoughts

Sharing data via the BitTorrent protocol can be an excellent way for universities, professionals, or even hobbyists to collaborate while minimizing costs and reducing load on servers. By distributing files across multiple peers and seeders, the burden on a single server is drastically reduced, and download speeds can improve for everyone involved. However, there are several risks to consider before diving in. Torrenting is not without its potential hazards - a little bit of computer literacy (as well as caution) can go a long way in ensuring a safe and effective experience.

For instance, if just one or two others find your data valuable and decide to download it, they can become seeders themselves, further distributing the load. This helps not only with bandwidth management but also increases the reliability of file transfers, as each downloader contributes to the overall availability of the file. However, you must ensure that the torrent is securely configured, as malicious actors or corrupted files can pose serious security threats. By taking simple precautions, such as validating the source and using a trusted tracker, you can reduce these risks.

In the right context, torrenting can be a powerful tool for file sharing and collaboration and the more institutions that recognize the benefits of this approach then the more efficient and viable it will become. However, it’s important to weigh the potential downsides, such as dealing with a lack of seeders or ensuring the privacy of your data. With thoughtful planning and the right precautions, torrenting can be a valuable and low-cost option for distributing large datasets.