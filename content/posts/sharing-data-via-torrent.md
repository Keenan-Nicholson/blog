+++
date = '2025-02-12'
draft = true
title = 'Sharing Big Data via Torrent Clients'
+++
## Introduction

My day job involves working with large datasets, and I often need to share terabytes of data with coworkers. That’s all well and good—provided your company doesn’t mind paying egress fees, sharing network folders, or swapping external hard drives. A few months ago, I needed to adapt some existing code for our use case. Before I could do that, I had to get it running with the original dataset which was just over a terabyte  in size.

This is where I found the inspiration for this article! The code, which was part of a graduate thesis for a student at Oregon State University, linked to a page with instructions to download datasets larger than 5GB via the BitTorrent Protocol.

>Datasets larger than 5GB deposited in ScholarsArchive@OSU will be distributed using the BitTorrent protocol. BitTorrent, in essence, splits the file into smaller pieces, distributes the smaller pieces to you, and then reconstructs them to recreate the whole - all in a way that reduces impact on our computer networks. In addition, each “piece” of the file carries a marker that helps verify the validity and integrity of the final file.

I thought this was a super interesting way to share large datasets in an academic setting so I decided to do some more digging.

## Why use a Torrent Client to Share Data?

<details>
  <summary>Relevant XKCD ⬇️</summary>
  <img src="https://imgs.xkcd.com/comics/file_transfer.png" alt="XKCD File Transfer">
</details>

As mentioned by OSU above, using a torrent client break the files into small pieces and then reassembles them on the users end, but how does this reduce the impact on the networks?

To understand how this may impact a network, we first need to discuss how torrenting works in a bit more detail. <a href="https://medium.com/@edouard.courty/how-does-the-torrent-protocol-work-4ff40615d2ba" target="_blank">This Medium article</a> by Edouard Courty offers a great explanation, in essence when you download a file through Torrent, you’re not getting it from a single server but from multiple sources, often referred to as __seeds__ and __peers__. When you initiate a download, your Torrent client will reach out to a tracker, which helps it find other users who have the same Torrent file downloaded and are currently sharing it (seeds) or are currently downloading the same content (peers). If this is a bit confusing, take a look at this <a href="https://www.reddit.com/r/explainlikeimfive/comments/10bl0sb/eli5_how_do_torrents_work/" target="_blank">reddit post</a> by u/Dekkars in ELI5.



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

This comes with some advantages: first, with a torrent, once the first few people start downloading, they also become seeders, meaning later downloaders can get pieces of the file from multiple sources instead of relying solely on your network. This has potential to vastly reduce the outgoing bandwidth over direct downloading, especially if you multiple people are trying to download the data simultaneously.

Next, it can increase the reliability of downloads. If the data server goes offline, you can still collect file parts from seeders and peers, whereas with a direct download, a server crash may cause the download to fail. Speaking of not relying on one server, another advantage of decentralization is sharing the load among all users participating in the torrent. This can reduce costs and help reduce the effects felt by server limitations.

Finally, torrenting has great scalability. If we have a bunch of people trying to direct download from the data server at the same time this can put immense stress on the network slowing the downloads to a crawl. However, when torrenting, each downloader is sharing bandwidth and reducing the stress on any one server.

Amazing! We have:

1. Reduced Bandwidth Strain on Your Network,
2. Faster Download Speeds,
3. More Reliable Downloads,
4. Decentralization and,
5. Better Scalability

why would we ever _not_ use a torrent client to share our data then?

## Sharing is Caring
