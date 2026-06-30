---
title: ECOWAS Cybersecurity Hackathon 2026 My Recap
date: 2026-06-30 00:00:00 +0000
categories: [ECOWAS, CTF]
tags: [ctf, cybersecurity, ecowas, hackathon, jeopardy, koth, red-vs-blue, nigeria]
math: false
mermaid: false
image:
  path: https://h4ckyou.github.io/assets/posts/2026-06-24-Ecowas-Recap/preview.png
---

It's been about a week since I got back and I've been putting off writing this, but here we go.

This is my full recap of the **ECOWAS Cybersecurity Hackathon 2026**, the regional finals held in Accra, Ghana, where Nigeria's **error** team finally climbed to the top of the podium.

![All teams lined up](https://h4ckyou.github.io/assets/posts/2026-06-24-Ecowas-Recap/IMG_4284.jpg)

## Who Am I

I'm Theory. Cybersecurity, CTFs, the whole thing. I'm Nigerian but for the past 8 months I've been based in Dakar, Senegal working as a cybersecurity analyst. I compete with team **error** alongside Mark ([h4cky0u](https://h4ckyou.github.io)), securedviki, and proflamyt. We've been at this together for years, and this was our fourth straight shot at the ECOWAS regional finals.

## Our ECOWAS History

Every year we've shown up. Every year we've been on the podium. But we hadn't won it yet.

| Year | Placement |
|------|-----------|
| 2022 | 🥈 2nd Place |
| 2023 | 🥉 3rd Place |
| 2024 | 🥉 3rd Place |
| 2026 | 🥇 1st Place |

Four years of podium finishes. 2026 was finally the one that mattered most.

## Qualification

Before the flights, you earn the right to be there. The Nigerian qualifying rounds ran across three phases: Jeopardy, King of the Hill, and Battleground, and **error** came in 1st across all three. That performance is what sent us to Accra.

## June 7: Dakar → Lomé → Accra

![Departure](https://h4ckyou.github.io/assets/posts/2026-06-24-Ecowas-Recap/IMG_4342.jpg)

Everyone else on the team flew out of Lagos. My June 7 looked completely different.

Being in Dakar meant I was routing to Ghana from a different starting point entirely. But what made the travel day genuinely memorable wasn't the logistics, it was the company.

I departed Dakar alongside the **Senegal national team (Jambars)**. Same flight. Same destination. Opposing sides in the competition. We talked at the gate, kept things light, and tried not to spend too much energy scoping each other out before we even got there. Good people, genuinely.

The route took us through **Lomé, Togo** for a layover, and that transit turned into something I didn't expect. Waiting in the same terminal were the **Benin team (Escadron)** and the **Togo team (RedTeam-TG)**, both also on their way to Accra. So for a stretch of time, what would end up being the top three teams on the final podium were all sitting in the same airport, none of us knowing any of that yet, sharing snacks and laughing at the coincidence of it all.

We boarded the onward flight together and landed in Accra that evening.

At the hotel, I finally linked up with the rest of my team. Mark and I hadn't seen each other in about 8 months, since I moved to Senegal. That reunion had a proper weight to it.

![Reunited in Accra](https://h4ckyou.github.io/assets/posts/2026-06-24-Ecowas-Recap/IMG_4302.jpg)

## June 8: Official Briefing

The next morning opened with the official briefing. The most important announcement came early: the scoreboard would remain **hidden** throughout the entire competition. No live rankings. No checking where you sat relative to other teams. You'd compete blind and trust that the work was accumulating.

That changes the psychology of the whole thing. You can't adjust your strategy based on the standings if you can't see them. You just have to execute, every phase, every session.

![Briefing](https://h4ckyou.github.io/assets/posts/2026-06-24-Ecowas-Recap/IMG_4321.jpg)

The Director of Digital Economy at the ECOWAS Commission was present and ran the session. Twelve nations. Six phases. One champion. That was the frame.

## The Competition: Six Phases

### Phase 1 & 6: Jeopardy

![Jeopardy phase](https://h4ckyou.github.io/assets/posts/2026-06-24-Ecowas-Recap/IMG_4326.jpg)

Jeopardy opened and closed the competition, traditional CTF categories, timed submissions, cumulative points. The energy in the room during the first round was noticeably different from prior years. First bloods were dropping at a pace I hadn't seen before.

The reason was obvious: AI-assisted workflows were in use across multiple teams. The nature of this competition has shifted. It's no longer purely about who knows the most, it's about who has built the sharpest workflow, who can tell when the LLM is reliable and when it's going to hallucinate you sideways. The challenges that genuinely separated teams were the ones that required human intuition the AI couldn't shortcut.

The first round bled into the early hours of the morning. We kept pushing.

### Phase 2: King of the Hill

![King of the Hill scoreboard](https://h4ckyou.github.io/assets/posts/2026-06-24-Ecowas-Recap/koth.png)

This is the phase where I felt the confidence lock in.

The format was two sessions, four teams per hill. Our first session placed us against **CYCLONE (Ghana)**, **Jambars (Senegal)**, the same crew I'd been traveling with the previous day, and **Sudo-SL (Sierra Leone)**.

We got into the target machine first. Escalated privileges. Patched the vulnerability to lock the others out. Maximum points, 1000, secured in that session.

But we didn't stop there. We automated the full exploit-and-patch cycle, so that at every hourly reset, we were back in immediately. It wasn't enough to be first once, the points come from sustained control. We ran it clean through every cycle.

![KotH session](https://h4ckyou.github.io/assets/posts/2026-06-24-Ecowas-Recap/IMG_4346.jpg)

### Phase 3: Code Review

Teams analyzed projected source code and called out vulnerabilities in real time. The examples ranged from C code to web applications: SQL injection, XSS, buffer overflow, and a PHP object deserialization vulnerability.

![Code review round](https://h4ckyou.github.io/assets/posts/2026-06-24-Ecowas-Recap/IMG_4347.jpg)

This was genuinely enjoyable. AI tooling is less useful when you're reading specific code projected on a screen in a room with 12 teams watching. You reason through it in real time with what you know. It felt like a purer test of foundational knowledge, and it was a welcome change of pace from the AI-augmented Jeopardy rounds.

### Phase 4: Red vs Blue

Two sessions, attacking and defending simultaneously. Our opponents were **SilySec (Guinea)**.

We drew first blood. Got into their system before they reached ours. But we were slower on patching our own side than we should have been, and they managed a counterattack before we sealed things up. It wasn't a clean run, but we absorbed it and held position.

![Red vs Blue](https://h4ckyou.github.io/assets/posts/2026-06-24-Ecowas-Recap/IMG_4455.jpg)

### Phase 5: Battleground

Speed format. Flag retrieval race against **CyberSharks (Cape Verde)**.

We captured the **user flag** first, clean execution. The root flag race was tighter. They got there ahead of us. It stung in the moment, but in the context of six phases where we'd been stacking consistent points, a single phase loss wasn't going to define the result.

![Battleground phase](https://h4ckyou.github.io/assets/posts/2026-06-24-Ecowas-Recap/IMG_4456.jpg)

## Waiting for the Results

After the final Jeopardy phase closed, the waiting started.

With a hidden scoreboard all week, nobody knew where they stood going into the closing ceremony. That uncertainty was real. You could feel it in the room.

![Closing ceremony](https://h4ckyou.github.io/assets/posts/2026-06-24-Ecowas-Recap/IMG_4482.jpg)

When the final standings came up:

| Rank | Country | Team |
|------|---------|------|
| 🥇 1st | Nigeria | error |
| 🥈 2nd | Benin | Escadron |
| 🥉 3rd | Togo | RedTeam-TG |

Nigeria. First place.

I kept thinking about that transit lounge in Lomé, me, the Benin guys, the Togo guys, all of us heading to the same competition with no idea we'd be standing on that exact podium in that exact order days later. That's the kind of thing that sticks with you.

![Winners](https://h4ckyou.github.io/assets/posts/2026-06-24-Ecowas-Recap/preview2.png)

![Team error with trophy](https://h4ckyou.github.io/assets/posts/2026-06-24-Ecowas-Recap/IMG_4500.PNG)

![All participants group photo](https://h4ckyou.github.io/assets/posts/2026-06-24-Ecowas-Recap/IMG_4503.PNG)

## The People

The competition is why you travel. Meeting the community is why you stay.

I'd been interacting online with people across the West African security scene for years, W1z4rd from Escadron (Benin), McSam, ka3n1x, troylynx, phoenix, and others, and this week was the first time I actually met most of them in person. W1z4rd had authored some of the challenges we were solving. Putting faces to names after that many online interactions is a specific kind of good.

![Meeting the community](https://h4ckyou.github.io/assets/posts/2026-06-24-Ecowas-Recap/charles.jpg)

The Cyber Security Authority Ghana organized a tour for all competing teams after the competition wrapped up. Good way to decompress and actually talk to people you'd been competing against all week as human beings instead of opponents.

![CSA Ghana tour](https://h4ckyou.github.io/assets/posts/2026-06-24-Ecowas-Recap/IMG_6144.jpg)

![Tour continued](https://h4ckyou.github.io/assets/posts/2026-06-24-Ecowas-Recap/IMG_6170.jpg)

## Everyone Together

![Group photo, all twelve nations](https://h4ckyou.github.io/assets/posts/2026-06-24-Ecowas-Recap/IMG_4336.jpg)

Twelve countries. One week. West Africa's cybersecurity community at its best.

## One More

![Me](https://h4ckyou.github.io/assets/posts/2026-06-24-Ecowas-Recap/meee.jpeg)

I flew out of Dakar in 2026 as part of the team that won. The geography was different from every prior year. The result was different too.

What didn't change: the team, the grind, the belief that putting in consistent work compounds over time. Four years of podium finishes. This is what the fourth year looks like when you've been preparing right.

Shoutout to Mark, securedviki, and proflamyt, this was built over years, not days. Shoutout to ECOWAS and the challenge authors for running a genuinely competitive event. Shoutout to every team that showed up and made this hard.

And to the Jambars: great travel companions. You'll get your year.

See everyone next time.

Theory
