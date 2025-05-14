---
date: '2025-02-13T00:00:00-06:00'
title: 'So you want to run untrusted code?'
---

Congrats, you just purchased a brand new domain name for your enterprising SaaS startup. Overnight you came up with the brilliant idea to start a company that runs other people’s code as a business. I applaud you on your originality. The only problem is… how are you going to do it **securely**?

Fortunately for you, I've spent a little bit of time (read: a few hours) over the last couple of days reading about how other people run other _other_ people’s code . Let’s see what your options are in 2025.

## Virtual Machines
Ah yes, good ol’ virtual machines. A tried-and-true method. The stable work horse of running code securely. They’re slow to start, resource intensive, but pretty darn secure — sort of like a Tank class in your favorite game.

## Containers
This is a really popular way to run other people’s code. The focus of my reading was not on this due to it’s ubiquity. Running containers securely seems like mostly a [solved problem](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/#:~:text=Use%20containers-,securely,-Docker%20restricts%20and). Probably analagous to a standard DPS class — faster than a tank, less resource intensive, pretty secure, not especially skilled in any one area. Moving on…

## V8 Isolates
Now we’re getting into some interesting territory. [V8](https://v8.dev/) is Google’s JavaScript and <abbr title="WebAssembly">Wasm</abbr> engine. My understanding is that every Chrome tab runs in its own isolate. Isolates have also become a popular way to run untrusted code, particularly because they have a _much_ lower overhead than virtual machine and start faster (~5 ms cold start).

> Isolates represents an isolated instance of the V8 engine. V8 isolates have completely separate states.

To force continuing the character-class metaphor, isolates are kind of like a Mage class — their ability to raise a billion low-health skeletons to to fight is pretty impressive.

### Deno Deploy
[Deno Deploy](https://deno.com/deploy) runs “serverless” JavaScript for its customers. It uses isolates to run its deployments. Deno Deploy runs each isolate in it’s own process for added security. It also runs the isolates with [Deno](#deno) (not surprsising given the name).

### Cloudflare Workers
[Cloudflare Workers](https://workers.cloudflare.com/) also runs “serverless” code for its customers. They support more languages than just JavaScript. Cloudflare has also open sourced it’s server runtime, [`workerd`](https://github.com/cloudflare/workerd). Workers also use isolates, _sometimes_ in separate processes. Cloudflare only runs isolates in the same processe because each process requires more resources and they want to keep costs low for customers. They’ll run an isolate   in its own process when it looks suss. They also run _every_ worker on _every_ node. How’s that for distributed computing?

> Sometimes, though, we do decide to schedule a worker in its own private process. We do this if it uses certain features that we feel need an extra layer of isolation.


<aside>
<div>
  <strong id="defense-in-depth">Defense-in-Depth</strong>
  <div>
    Something that both Deno Deploy and Cloudflare Workers make mention of is the concept “defense-in-depth”. Defense-in-depth means having multiple layers of security against would-be attackers. In practice, this means not just using isolates, but also using <a href="https://man7.org/linux/man-pages/man7/namespaces.7.html">namespaces</a>, <a href="https://man7.org/linux/man-pages/man7/cgroups.7.html">cgroups</a>, and <a href="https://man7.org/linux/man-pages/man2/seccomp.2.html">seccomp</a> (the last of which I had never heard of before).
  </div>
</div>
</aside>


## Deno
  Deno is a JavaScript/TypeScript runtime. Deno by itself has some cool security features. By default it disables file system access, network connectivity, and more. We already mentioned Deno above, but there is one more product that uses Deno that is worth mentioning. 

<aside>
<span>If you’re interested in how Deno compares with Node, check out this presentation from the creator of Deno… and Node: <a href="https://www.youtube.com/watch?v=M3BM9TB-8yA">10 Things I Regret About Node.js</a>.
</span>
</aside>

### Val Town
[Val Town](https://www.val.town/) is a “collaborative website to write and deploy TypeScript”. Val Town has had a interesting evolution, from briefly using Node.js with it’s [`vm`](https://nodejs.org/api/vm.html)/[`vm2`](https://github.com/patriksimek/vm2) modules, to using just Deno, to using Node + Deno. As of February 2025, Val Town runs customer’s code in Deno as a subprocesses of a Node server using [`node-deno-vm`](https://github.com/casual-simulation/node-deno-vm), a Node -> Deno virtual machine module.

## microVMs
Alas, we’ve come full circle. Everthing that is old is new again, and so VMs have resurfaced. microVMs are virtual machines that are purpose-built for running workloads securely with a smaller footprint than a traditional VM. Their cold boot time is in the range of 200–300 ms. I believe the term “microVM” was coined by AWS, the creators of Firecracker.

<aside>
<span>
I’m not 100% sure if it is consider a microVM, but <a href="https://gvisor.dev/">gVisor</a> is in the same vein as Firecracker. I'm not very familar with it, but they have some <a href="https://gvisor.dev/users/">heavy hitters on their users page. It was created by Google for GCP.
</span>
</aside>

[Firecracker](https://firecracker-microvm.github.io/) was created by AWS for use with AWS Lambda and Fargate. Firecracker microVMs use the “Linux Kernel-based Virtual Machine” (KVM… no, not _that_ <abbr title="Keyboard, Video, Mouse">KVM</abbr>). I’m not very familar with KVM, nor QEMU, which Firecracker is compared to. microVMs are like a Rogue class — they’re small but pack a punch.

Firecracker is also used by cool new companies like…

### Fly.io
[Fly.io’s](https://fly.io/) excellent blog is where I first read about Firecracker. Honestly, just go read these two posts from them on how they’re using it. The first link is a much better version of this article.

- [Sandboxing and Workload Isolation ](https://fly.io/blog/sandboxing-and-workload-isolation/)
- [Docker without Docker](https://fly.io/blog/docker-without-docker/)

## WebAssembly (Wasm)
Running [Wasm](https://webassembly.org/) in the cloud is a [thing](https://wasmcloud.com/.) I’ve never done it and don’t know much about it. Smarter people than me have [written about it](https://kevinhoffman.blog/post/wasmcloud_unisoncloud/.)

## Summary
So you came here expecting some insightful synopsis or a spicy hot-take? Well, based on my extreme expertise (read again: _hours_ of reading /s), I would suggest one of two approaches, depending on your scenario:

- Running functions-as-a-service: isolates + Deno + [extra security sauce](#defense-in-depth)
- Running compute-as-a-service: Firecracker

I know, _shocking_ conclusions I came to, eh? Thanks for making it this far. You’ve earned a screenshot of @mrkurt[^1] replying to a Hacker News [post](https://news.ycombinator.com/item?id=31740885) about V8 isolates, with a response from @kentonv[^2]:

{{< figure
  src="/images/2025-02-13-isolates-hn.png"
  link="https://news.ycombinator.com/item?id=31759170"
>}}

and a fancy product comparison table:

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Method</th>
      <th>Processes</th>
      <th>Other Security</th>
      <th>Language</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Deno Deploy</th>
      <td>Isolates + Deno</td>
      <td>Always separate</td>
      <td>

Hypervisor resource monitoring (WatchDog), `namespaces`, `seccomp`, `cgroups`

</td>
      <td>Rust 🦀</td>
    </tr>
    <tr>
      <th>Cloudflare Workers</th>
      <td>Isolates</td>
      <td>Sometimes separate</td>
      <td>"cordons" (multiple runtime instances), `namespaces`, `seccomp`, no network access (only Unix sockets)</td>
      <td>

[C++?](https://github.com/search?q=repo%3Acloudflare%2Fworkerd++language%3AC%2B%2B&type=code)

</td>
    </tr>
    <tr>
      <th>Val Town</th>
      <td>Deno</td>
      <td>Subprocesses + VM</td>
      <td>

Deno VM wrapper, [`cgroups`?](https://macwright.com/2025/12/15/recently#:~:text=it%20a%20pleasant-,tool.,-Frankly%2C%20with%20every)

</td>
      <td>TypeScript/Rust?</td>
    </tr>
    <tr>
      <th>Fly.io</th>
      <td>Firecracker</td>
      <td>…it’s a VM</td>
      <td>?</td>
      <td>Rust</td>
    </tr>
    <tr>
      <th>wasmCloud</th>
      <td>Wasm</td>
      <td colspan="3" style="text-align: center;">¯\_(ツ)_/¯</td>
    </tr>
  </tbody>
</table>

## Resources
- V8 Isolates
  - [The V8 Sandbox](https://v8.dev/blog/sandbox)
- Containers
    - [Understanding Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
- [Deno](https://deno.com/)
    - [Security and permissions](https://docs.deno.com/runtime/fundamentals/security/)
    - [Ask HN: Pros and cons of V8 isolates?](https://news.ycombinator.com/item?id=31740885)
- [Deno Deploy](https://deno.com/deploy)
    - [The Anatomy of an Isolate Cloud](https://deno.com/blog/anatomy-isolate-cloud#the-runner-as-an-isolate-hypervisor)
    - [How security and tenant isolation allows Deno Subhosting to run untrusted code securely](https://deno.com/blog/subhosting-security-run-untrusted-code)
    - [How we built a secure, performant, multi-tenant cloud platform to run untrusted code](https://deno.com/blog/build-secure-performant-cloud-platform)
- [Cloudflare Workers](https://workers.cloudflare.com/)
    - [Mitigating Spectre and Other Security Threats: The Cloudflare Workers Security Model](https://blog.cloudflare.com/mitigating-spectre-and-other-security-threats-the-cloudflare-workers-security-model/)
    - [Cloud Computing without Containers](https://blog.cloudflare.com/cloud-computing-without-containers/)
    - [Introducing workerd: the Open Source Workers runtime](https://blog.cloudflare.com/workerd-open-source-workers-runtime/)
- [Fly.io](https://fly.io/)
    - [Sandboxing and Workload Isolation ](https://fly.io/blog/sandboxing-and-workload-isolation/)
    - [Docker without Docker](https://fly.io/blog/docker-without-docker/)
- [Val Town](https://www.val.town/)
    - [The first four Val Town runtimes](https://blog.val.town/blog/first-four-val-town-runtimes/)
- Wasm
    - [wasmCloud and Unison Cloud](https://kevinhoffman.blog/post/wasmcloud_unisoncloud/)

[^1]: [@mrkurt](https://x.com/mrkurt) — founder and CEO of Fly.io
[^2]: [@kentonv](https://x.com/kentonvarda) — tech lead of Cloudflare Workers
