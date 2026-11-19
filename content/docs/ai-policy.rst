---
title: "Ashen and Yanartas Policy on AI-assisted Contributions"
---

Ashen and Yanartas Policy on AI-assisted Contributions
======================================================

The Ashen and Yanartas projects have the following policies on acceptable AI use.

AI Use
------

 * If you enjoy creating the thing, consider doing it yourself and not using AI.
 * AI-generated raster graphics are entirely unacceptable. AI can never create art, only people can.
 * AI-generated code is *welcome* within the Ashen software projects, but *unwelcome* within the Yanartas hardware
   projects. 
 * AI-generated code that is only used once, or a few times, and whose output is non-critical e.g. because it is used
   under direct human supervision may be "vibe" coded.
 * For all other code, and especially for long-running code where correctness is important such as in ashen-firmware,
   close human oversight and review of every single line of code is strictly necessary. All such code that is published
   *must* be properly understood. When using an LLM to generate or modify such code, utmost caution must be placed on
   keeping it simple, clean, maintainable, and human-readable.
 * In general, documentation and other prose should never be generated using AI, and for editing AI should only be used
   with heightened caution. Exceptions to this rule are laborious and boring tasks such as formatting a markdown table,
   or synchronizing pinout information between a schematic and a documentation file. AI as a translation aid for
   non-native speakers can be an exception here, but with a lot of caution applied. We want to see your own style shine
   through in your writing, not an AI-homogenized version of the mean of all English.
 * A consequence of the "no prose" rule above is that any AI-written communication such as PR descriptions or issues are
   strictly unwelcome.
 * AI-assisted auxiliary tasks such as debugging and code and design review are acceptable both within the Ashen and
   Yanartas projects.
 * AI-generated translations of public documentation are acceptable only if they are reviewed and edited by a competent
   speaker.
 * Due to the intense resource footprint of AI tools, any use of AI must be justified by a proportional saving in either
   human labor, or in CO2 footprint due to reduced iteration cycles.
 * All hardware designs *should* be reviewed using AI before production due to the chance to catch resource-intensive
   failed prototype runs.

For details on the ecological angle, have a look at the Yanartas LLM policy document `here
<https://git.yasec.de/yasec/yanartas-sensors/src/branch/main/llm-policy.md>`__. There, we did a back-of-the-envelope
calculation of carbon footprint of using AI for a complex design review, and of a failed prototype run, and the score
is roughly 1.5 kg for the review vs. 25 kg for a failed prototype run.

AI Disclosure
-------------

 * Any substantially AI-generated code must be disclosed as such, e.g. in a source code comment and/or or through a
   standard commit message trailer. The goal is that humans reading code should always be aware of whether the code is
   AI-generated or not.
 * The rare AI-generated section in documentation should be prefaced with a note to the reader.
 * Any AI-generated code should be labelled with some information on its author's confidence in the resulting code, such
   as "human written, enjoy" or "slop-generated don't read, cognitohazard".

