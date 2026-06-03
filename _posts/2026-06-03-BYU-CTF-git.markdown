---
layout: post
title:  "Git Challenges"
date:   2026-06-03 05:46:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /BYU-CTF-2026-git/
---
* TOC
{:toc}

## Gitastic 1

**Category**: Git

**Author**: Zinko

**Description**: We've recently found out that one of our employees has been exfiltrating information to a competitor. They did it via messages attached to each code change he made. Find the secret that was exfiltrated! `git clone git://chals.cyberjousting.com:1363/challenge`

I clone the repo:
```shell
git clone git://chals.cyberjousting.com:1363/challenge  
Cloning into 'challenge'...  
hint: Using 'master' as the name for the initial branch. This default branch name  
hint: is subject to change. To configure the initial branch name to use in all  
hint: of your new repositories, which will suppress this warning, call:  
hint:  
hint:   git config --global init.defaultBranch <name>  
hint:  
hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and  
hint: 'development'. The just-created branch can be renamed via this command:  
hint:  
hint:   git branch -m <name>  
remote: Enumerating objects: 7252, done.  
remote: Counting objects: 100% (7252/7252), done.  
remote: Compressing objects: 100% (2566/2566), done.  
remote: Total 7252 (delta 4678), reused 7252 (delta 4678), pack-reused 0 (from 0)  
Receiving objects: 100% (7252/7252), 1.90 MiB | 4.57 MiB/s, done.  
Resolving deltas: 100% (4678/4678), done.
```

I dump all commit messages:
```shell
git log --pretty=format:"%B" --reverse > all_commit_messages.txt  

cat all_commit_messages.txt | wc -l  
4732  

cat all_commit_messages.txt | grep byu  
stochastic windowing algorithm of the dynamic syntax parser. While this condition presented no immediate fault state, it introduced an untenable degree of epistemological dissonance concerning the repositorys self-descriptive coherence, necessitating a Hyper-Dimensional Meta-Structural Integrity Assertion for Latent Information Architecture Refraction. 1. Lexical Substrate De-Entanglement Protocol All introductory and concluding descriptive passages within the main INTERFACE_ABSTRACT.md and related secondary module support documents have undergone a comprehensive transparency inversion cycle. This involved the precise manipulation of appositive clauses and the regularization of semicolons to guarantee perfect topological isomorphism with the established cognitive baseline. This ensures the Hyper-Dimensional Meta-Structural Integrity Assertion for Latent Information Architecture Refraction now precisely mirrors the reciprocal relational topology of the applications intended operational continuum. 2. Configuration Relational Calculus Re-Evaluation Within the internal .log and supplemental .map files, explanatory notations pertaining to dormant structural affordances (specifically those governing the Event Horizon Data Mirror) have been systematically relocated to an isolated, non-traversable partition. This action facilitates the Hyper-Dimensional Meta-Structural Integrity Assertion for Latent Information Architecture Refraction by pre-emptively neutralizing any potential future conflict states during the hypothetical temporal divergence phase of the deployment manifold, despite the fact that the feature was formally suspended in the tertiary deployment window. 3. Deployment of the Unitary Affordance Anchor (.UAA). Wait a minute... theres a secret message here! byuctf{I_hop3_y0u_didnt_s3@rch_manually}. Great job on finding it. Im going to go back to posting now... his extensive revision introduces an imperative, yet obliquely peripheral, reassessment of the implicit ontological tethering between the systems operational parameters and the repositorys collateral, non-executable informational projections. The fundamental motivation is not to alter any functional behavior, but rather to establish a Hyper-Dimensional Meta-Structural Integrity Assertion for Latent Information Architecture Refraction, designed to counteract the continuous informational decoherence inherent in high-volume, human-generated contextual  
data. We are, in essence, fortifying the informational nucleus against semantic entropy. The impetus for this pre-emptive structural normalization stemmed from an observed non-zero probability state where the intrinsic complexity of specific YAML schema arrays was not optimally reconciled with the stochastic windowing algorithm
```

`byuctf{I_hop3_y0u_didnt_s3@rch_manually}`

___

## Gitastic 2

**Category**: Git

**Author**: Zinko

**Description**: Some mysterious person made a commit to our repo! Who's the odd one out? 
`git clone git://chals.cyberjousting.com:1363/challenge`

```shell
git shortlog -s -n -e  
   387  Zinko <zinkogamez@gmail.com>  
   387  sawyer <zinkogamez@gmail.com>  
   387  walker <zinkogamez@gmail.com>  
   387  walter <zinkogamez@gmail.com>  
   387  william <zinkogamez@gmail.com>  
   387  wyatt <zinkogamez@gmail.com>  
     1  byuctf{wh0s_th3_auth0r?} <zinkogamez@gmail.com>  
     1  zinkozapper <zinkogamez@gmail.com>
```

- `-s`: suppresses the commit descriptions and just shows the count
- `-n`: sorts the authors by the number of commits (highest to lowest)
- `-e`: shows the email addresses associated with the authors

`byuctf{wh0s_th3_auth0r?}`

___

## Gitastic 3

**Category**: Git

**Author**: Zinko

**Description**: With the increased growth of our startup we've decided that we should start tagging our code releases! However we accidentally tagged one with the wrong thing, can you find it? `git clone git://chals.cyberjousting.com:1363/challenge`

```shell
git tag -n | head -n 35  
v1.0            Init commit  
v1.0.1          Pre-emptive Synchronicity Layer Re-Calibration for Non-Source-Code Artifact Contextualization Flux  
v1.0.10         Pre-emptive Synchronicity Layer Re-Calibration for Non-Source-Code Artifact Contextualization Flux  
v1.0.100        Tag Count #100: v1.0.100  
v1.0.101        Tag Count #101: v1.0.101  
v1.0.102        Tag Count #102: v1.0.102  
v1.0.103        Tag Count #103: v1.0.103  
v1.0.104        Tag Count #104: v1.0.104  
v1.0.105        Tag Count #105: v1.0.105  
v1.0.106        Tag Count #106: v1.0.106  
v1.0.107        Tag Count #107: v1.0.107  
v1.0.108        Tag Count #108: v1.0.108  
v1.0.109        Tag Count #109: v1.0.109  
v1.0.11         Pre-emptive Synchronicity Layer Re-Calibration for Non-Source-Code Artifact Contextualization Flux  
v1.0.110        Tag Count #110: v1.0.110  
v1.0.111        Tag Count #111: v1.0.111  
v1.0.112        Tag Count #112: v1.0.112  
v1.0.113        Tag Count #113: v1.0.113  
v1.0.114        Tag Count #114: v1.0.114  
v1.0.115        Tag Count #115: v1.0.115  
v1.0.116        Tag Count #116: v1.0.116  
v1.0.117        Tag Count #117: v1.0.117  
v1.0.118        Tag Count #118: v1.0.118  
v1.0.119        Tag Count #119: v1.0.119  
v1.0.12         Pre-emptive Synchronicity Layer Re-Calibration for Non-Source-Code Artifact Contextualization Flux  
v1.0.120        Tag Count #120: v1.0.120  
v1.0.121        Tag Count #121: v1.0.121  
v1.0.122        Tag Count #122: v1.0.122  
v1.0.123        Tag Count #123: v1.0.123  
v1.0.124        Tag Count #124: v1.0.124  
v1.0.125        Tag Count #125: v1.0.125  
v1.0.126        Tag Count #126: v1.0.126  
v1.0.127        Tag Count #127: v1.0.127  
v1.0.128        Tag Count #128: v1.0.128  
v1.0.129        sdasdkjgarjknakfndlvoaiutiqewrhqiwerqjewnqwebehfbwhbqeuorueoqrhqoewuirhnbadnabyuctf{that's_a_lot_of_tags_wow}bkmsldfvnjsdfgkjwhvdsklfvsjlfbgwglkjhjfvkg sfgjnoeigudshfwjlqfwebekjbfhdav
```

`-n` lists tags along with messages. 

`byuctf{that's_a_lot_of_tags_wow}`

___

## Gitastic 4

**Category**: Git

**Author**: Zinko

**Description**: Walker reported that he accidentally pushed an API key to our public repo. Luckily he deleted it afterward so we're completed safe and secure! 
`git clone git://chals.cyberjousting.com:1363/challenge`

I run `git log -S "byuctf" -p`:

- `-S "byuctf"`: looks for commits where the number of occurrences of the string "byuctf" changed

- `-p`: shows the patch/diff of that commit

```shell
git log -S "byuctf" -p
commit f3361dcede9b2337bbbf51ff08d6fbb215186bbd
Author: walker <zinkogamez@gmail.com>
Date:   Thu Jan 29 10:45:28 2026 -0700

    Technical Update: Systemic Stabilization Report
    
    Hyper-Dimensional Meta-Structural Integrity Assertion for Latent Information Architecture Refraction
    
    This extensive revision introduces an imperative, yet obliquely peripheral, reassessment of the implicit ontological tethering between the system's operational parameters and the repository's collateral, non-executable informational projections. The fundamental motivation is not to alter any functional behavior, but rather to establish a Hyper-Dimensional Meta-Structural Integrity Assertion for Latent Information Architecture Refraction, designed to counteract the continuous informational decoherence inherent in high-volume, human-generated contextual data. We are, in essence, fortifying the informational nucleus against semantic entropy.
    
    The impetus for this pre-emptive structural normalization stemmed from an observed non-zero probability state where the intrinsic complexity of specific YAML schema arrays was not optimally reconciled with the stochastic windowing algorithm of the dynamic syntax parser. While this condition presented no immediate fault state, it introduced an untenable degree of epistemological dissonance concerning the repository's self-descriptive coherence, necessitating a Hyper-Dimensional Meta-Structural Integrity Assertion for Latent Information Architecture Refraction.
    1. Lexical Substrate De-Entanglement Protocol
    
    All introductory and concluding descriptive passages within the main INTERFACE_ABSTRACT.md and related secondary module support documents have undergone a comprehensive transparency inversion cycle. This involved the precise manipulation of appositive clauses and the regularization of semicolons to guarantee perfect topological isomorphism with the established cognitive baseline. This ensures the Hyper-Dimensional Meta-Structural Integrity Assertion for Latent Information Architecture Refraction now precisely mirrors the reciprocal relational topology of the application's intended operational continuum.
    2. Configuration Relational Calculus Re-Evaluation
    
    Within the internal .log and supplemental .map files, explanatory notations pertaining to dormant structural affordances (specifically those governing the 'Event Horizon Data Mirror') have been systematically relocated to an isolated, non-traversable partition. This action facilitates the Hyper-Dimensional Meta-Structural Integrity Assertion for Latent Information Architecture Refraction by pre-emptively neutralizing any potential future conflict states during the hypothetical temporal divergence phase of the deployment manifold, despite the fact that the feature was formally suspended in the tertiary deployment window.
    3. Deployment of the Unitary Affordance Anchor (.UAA)
    
    A novel, structurally minimalist zero-dimension .uaa artifact has been provisioned within the root of the data hierarchy. This construct serves no active role other than to function as a semiotic nexus for a future, yet-to-be-defined structural token intended to enforce Hyper-Dimensional Meta-Structural Integrity Assertion for Latent Information Architecture Refraction. It constitutes a necessary structural pre-conditioning against currently undefined stress vectors within the aggregated operational matrix.
    
    Summary
    
    In summary, this commit stands as a validation of the unceasing dedication to systemic clarity within the repository's auxiliary-context layer. It enforces stability upon the descriptive components through Hyper-Dimensional Meta-Structural Integrity Assertion for Latent Information Architecture Refraction, adjusts the probabilistic entanglement of configuration metadata, and preemptively mitigates high-order conceptual cascade failures arising from inherent volatility in the descriptive language.
    
    This convoluted message, which I promise I definitely wrote myself, exists to prove that complexity is merely a choice, not an inevitability!
    
    No secret message here, sorry!

diff --git a/apikey.txt b/apikey.txt
deleted file mode 100644
index b0b2f81..0000000
--- a/apikey.txt
+++ /dev/null
@@ -1 +0,0 @@
-byuctf{But_th3s_was_d3l3t3d?}

commit a2cf27401d41f6cdef010abe4feeede890352fd3
Author: walker <zinkogamez@gmail.com>
Date:   Thu Jan 29 10:36:30 2026 -0700

    [200~Hyper-Dimensional Meta-Structural Integrity Assertion for Latent Information Architecture Refraction
    
    This extensive revision introduces an imperative, yet obliquely peripheral, reassessment of the implicit ontological tethering between the system's operational parameters and the repository's collateral, non-executable informational projections. The fundamental motivation is not to alter any functional behavior, but rather to establish a rigorous, albeit intrinsically volatile, probabilistic stabilization field designed to counteract the continuous informational decoherence inherent in high-volume, human-generated contextual data. We are, in essence, fortifying the informational nucleus against semantic entropy.
    
    The impetus for this pre-emptive structural normalization stemmed from an observed non-zero probability state where the intrinsic complexity of specific YAML schema arrays was not optimally reconciled with the stochastic windowing algorithm of the dynamic syntax parser. While this condition presented no immediate fault state, it introduced an untenable degree of epistemological dissonance concerning the repository's self-descriptive coherence.
    1. Lexical Substrate De-Entanglement Protocol
    
    All introductory and concluding descriptive passages within the main INTERFACE_ABSTRACT.md and related secondary module support documents have undergone a comprehensive transparency inversion cycle. This involved the precise manipulation of appositive clauses and the regularization of semicolons to guarantee perfect topological isomorphism with the established cognitive baseline. This ensures the documentation now precisely mirrors the reciprocal relational topology of the application's intended operational continuum.
    2. Configuration Relational Calculus Re-Evaluation. Did you try to grep reverse search this? Rip lol.
    
    Within the internal .log and supplemental .map files, explanatory notations pertaining to dormant structural affordances (specifically those governing the 'Event Horizon Data Mirror') have been systematically relocated to an isolated, non-traversable partition. This action pre-emptively neutralizes any potential future conflict states during the hypothetical temporal divergence phase of the deployment manifold, despite the fact that the feature was formally suspended in the tertiary deployment window. This is purely an exercise in archival context remediation.
    3. Deployment of the Unitary Affordance Anchor (.UAA)
    
    A novel, structurally minimalist zero-dimension .uaa artifact has been provisioned within the root of the data hierarchy. This construct serves no active role other than to function as a semiotic nexus for a future, yet-to-be-defined structural token intended to enforce pan-dimensional configuration rendering consistency. It constitutes a necessary structural pre-conditioning against currently undefined stress vectors within the aggregated operational matrix.
    
    In summary, this commit stands as a validation of the unceasing dedication to systemic clarity within the repository's auxiliary-context layer. It enforces stability upon the descriptive components, adjusts the probabilistic entanglement of configuration metadata, and preemptively mitigates high-order conceptual cascade failures arising from inherent volatility in the descriptive language.
    
    This convoluted message, which I promise I definitely wrote myself, exists to prove that complexity is merely a choice, not an inevitability!
    
    No secret message here, sorry!

diff --git a/apikey.txt b/apikey.txt
index e69de29..b0b2f81 100644
--- a/apikey.txt
+++ b/apikey.txt
@@ -0,0 +1 @@
+byuctf{But_th3s_was_d3l3t3d?}
```

`byuctf{But_th3s_was_d3l3t3d?}`

___

## Gitastic 5

**Category**: Git

**Author**: Zinko

**Description**: We're certain that someone critical info in the secrets.txt file yet we can't see anything. It's like someone has replaced it with something else! We need you to get to the bottom of this mysterious case. `git clone git://chals.cyberjousting.com:1363/challenge`

For this one I need to manually force Git to pull down any replacement rules from the server:
```shell
git fetch origin 'refs/replace/*:refs/replace/*'  
remote: Enumerating objects: 4, done.  
remote: Counting objects: 100% (4/4), done.  
remote: Compressing objects: 100% (2/2), done.  
remote: Total 3 (delta 1), reused 0 (delta 0), pack-reused 0 (from 0)  
Unpacking objects: 100% (3/3), 351 bytes | 175.00 KiB/s, done.  
From git://chals.cyberjousting.com:1363/challenge  
 * [new ref]         refs/replace/f88c2adbe25705ad54cabfd0c84826235446ffd0 -> refs/replace/f88c2adbe25705ad54cabfd0c84826235446ffd0
```

Then I have the original file:
```shell
git replace -l  
f88c2adbe25705ad54cabfd0c84826235446ffd0  

git cat-file -p HEAD:secrets.txt  
byuctf{I_lov3_s3cr3t_files}
```

`byuctf{I_lov3_s3cr3t_files}`
