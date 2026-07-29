# Creative Analysis of AI's Carbon Footprint

*This is an essay from the same module as [the air-quality monitoring analysis](./ANALYSIS.md)
in this repo, written for the same Continuous Assessment. It's the essay text only. The
assignment cover sheet, student ID, and signature pages from the original submission
aren't included here.*

Most of the public debate about AI's environmental impact lands on one thing: the
training run. The story is always about the electricity used to train a large language
model to predict the next word, and the carbon that electricity produces. That's true, but
it's not the whole picture. Every GPU, every accelerator, every server rack that a model
gets trained on had to be built in a semiconductor fab first, and fabs consume enormous
amounts of energy, water, and chemicals before a single training run ever starts. In this
essay I look at AI's carbon footprint through Intel, a company that sits on both sides of
this problem: it runs the fabs that produce a lot of that upstream carbon, and it also
sells the AI accelerators that compete directly in this market. My argument is that AI's
carbon footprint has to include the whole hardware supply chain, not just the electricity
bill after the chips are deployed, and that Intel's recent track record shows how easily a
corporate climate commitment can get pushed aside once the company needs to compete on AI.

## Current impact analysis

Early research on AI's carbon footprint focused on operational energy, which makes sense
as a starting point. Strubell et al. (2019) produced one of the first real numbers here,
estimating that training a large Transformer model with neural architecture search
released around 626,155 pounds of CO2-equivalent, about five times the lifetime emissions
of an average American car including its production. Patterson et al. (2021) extended this
to several large models (T5, Meena, GShard, Switch Transformer, and GPT-3) and estimated
about 552 tonnes of CO2-equivalent for a single GPT-3 training run, with emissions varying
a lot depending on the data centre's energy mix and hardware efficiency. Luccioni et al.
(2023) looked at the same question from a lifecycle angle: BLOOM's training released about
24.7 tonnes of CO2-equivalent counting dynamic power use alone, but 50.5 tonnes once
equipment manufacturing and operational overhead are added in, roughly double. That gap is
the whole point. The three factors this module's brief points to, energy sourcing for data
centres, the computational load of training, and the hardware used to do it, show up in
both studies.

Hardware is the piece that gets the least attention, and I think it deserves more. Gupta et
al. (2020), in *Chasing Carbon: The Elusive Environmental Footprint of Computing*, make the
point that as computing gets more operationally efficient (better algorithms, more
renewable energy at data centres), the *share* of embedded carbon in computing actually
goes up, not down. That's where Intel fits in directly: semiconductor manufacturing is one
of the most resource-intensive manufacturing processes there is, using ultrapure water,
huge volumes of process chemicals, and constant high-purity climate control. To give a
sense of scale, Intel's own disclosures say it conserves about 10.5 billion gallons of
water a year and has reached net-positive water status in the United States, India,
Mexico, and Costa Rica (Intel Corporation, 2025). On electricity, Intel says 98 to 99% of
its electricity use now comes from renewable sources, and its Scope 1 and 2 emissions are
down 24% since a 2019 baseline (Intel Corporation, 2025).

Those headline numbers look less clean under a more critical read of the same period.
Scope 1 emissions intensity has actually gone up 56% over the last five years, and once you
factor in outsourced manufacturing at TSMC (about 30% of Intel's chip volume), actual
intensity may have more than doubled, which the renewable-electricity story doesn't
capture at all (Hansell, 2025). Real progress on operational electricity sourcing sitting
next to a worsening manufacturing-intensity picture is exactly what a training-run-only
view of AI's carbon footprint would completely miss, and it's made worse by Intel's
continued multi-billion dollar fab expansion in Arizona and Ohio, built specifically to
meet demand for the advanced process nodes AI needs (Intel Corporation, 2025).

## Future projections and mitigation strategies

If current trends keep going, AI's total carbon footprint is likely to grow a lot over the
next decade, both from the services people use and the hardware needed to run them. AI's
electricity use could reach the scale of a small country within a few years, somewhere
around 85 to 134 TWh annually by 2027, as inference workloads keep growing and start to
outpace training workloads in total energy use (de Vries, 2023). On the hardware side,
Intel's own construction schedule is a good signal of this: its Ohio fabs won't be ready
for production until 2030 to 2032, while its Arizona expansion, part of a $50 billion
regional investment, is aimed specifically at AI-relevant process nodes like Intel 18A and
20A (Intel Corporation, 2025). Every new fab is a large upfront embodied-carbon cost, years
before the chips it produces do any actual AI work, so if anything the hardware side of
AI's footprint is growing faster in the near term than the operational side.

Mitigation needs to happen on two fronts at once. On the operational side, Wu et al. (2022)
propose full-stack co-design, pairing more efficient model architectures with carbon-aware
scheduling, where computation gets scheduled for times and places where the grid is
cleaner. Intel's Gaudi 3 accelerator is one example of this at the hardware level: the
company says it uses around 40% less power than Nvidia's H100 on comparable inference
tasks, which directly cuts operational emissions per unit of AI output (Shilov, 2024).
Intel has also set a goal to improve product energy efficiency tenfold by 2030 for its
client and server processors, which would meaningfully cut the downstream (Scope 3)
emissions its customers generate just by using its chips (Intel Corporation, 2025).

On the manufacturing side, though, efficiency gains alone won't be enough. This needs
policy too. I'd propose two things here. First, standardised disclosure of embodied carbon
per chip, something like a nutrition label, so AI developers can factor that into
purchasing and design decisions instead of only optimizing for performance per watt.
Second, public funding for fab construction, including the US CHIPS and Science Act (which
is part of what's funding Intel's Arizona and Ohio expansion), should come with real
climate commitments attached, not just language, so the gap this essay describes between
what companies promise and what they actually do doesn't just keep repeating.

## Ethical considerations

Beyond the raw emissions numbers, there are ethical questions tied to where AI hardware
actually gets built. Fabs get built in specific communities, and their water and energy
demands compete directly with the needs of the people who live there. Intel's expansion in
Arizona and Ohio is happening in regions that already deal with water stress, so the
company's water-positive claims are as much about environmental justice as they are about
corporate messaging. The benefits of AI line up with SDG 6 (Clean Water and Sanitation),
SDG 7 (Affordable and Clean Energy), and SDG 13 (Climate Action), but those benefits get
spread globally while the resource costs land locally, often in communities that are
already vulnerable.

Intel's recent history is a good example of why this is hard. While cutting 25% of its
workforce and dealing with falling sales in the middle of an AI-chip race, Intel quietly
dropped its 2030 goal of a 30% cut in supplier emissions, replacing it with vaguer language
about a "mid-decade refresh" and a 2050 deadline, and the new CEO's compensation plan no
longer includes a greenhouse-gas reduction target at all (Hansell, 2025). One industry
analyst put it bluntly: "Unfortunately, in most companies, sustainability is an
afterthought, as profits are the top priority" (Hansell, 2025). I think that's the core
ethical problem here. Sustainability pledges made during good times don't mean much if
they get dropped the moment things get financially tight, and that's exactly the moment AI
companies racing to catch up are most tempted to cut corners. What's actually needed are
mechanisms that survive more than one business cycle: independent third-party
verification of both operational and embodied-carbon numbers, and climate conditions
attached to public subsidies, so commitments can't just be walked back whenever it's
convenient.

## Conclusion

AI's carbon footprint isn't just the electricity used to train and run models. It also
includes the carbon cost of the hardware that makes AI possible in the first place. Intel
is a good example of both sides of that: real progress on renewable electricity and water
use, an accelerator programme aimed at efficiency, and, at the same time, a willingness to
scale back manufacturing-emissions targets under business pressure. What AI actually needs
is a sustainable path that accounts for the full hardware lifecycle, standardised
embodied-carbon reporting, and accountability mechanisms strong enough to hold up against
the same business pressures that are driving AI's growth in the first place.

## References

De Vries, A. (2023). The growing energy footprint of artificial intelligence. *Joule*,
7(10), 2191 to 2194. https://doi.org/10.1016/j.joule.2023.09.004

Gupta, U., Kim, Y. G., Lee, S., Tse, J., Lee, H.-H. S., Wei, G.-Y., Brooks, D., & Wu, C.-J.
(2020). *Chasing Carbon: The Elusive Environmental Footprint of Computing*
(arXiv:2011.02839). arXiv. https://doi.org/10.48550/arXiv.2011.02839

Hansell, S. (2025, December 3). How Intel's sales tailspin sidelined its 2030
sustainability ambitions. *Trellis*.
https://trellis.net/article/intel-sales-tailspin-sidelined-2030-sustainability-ambitions/

Intel Corporation. (2025). *2024-25 Corporate Responsibility Report*. Intel Corporation.
https://csrreportbuilder.intel.com/pdfbuilder/pdfs/CSR-2024-25-Executive-Summary.pdf

Patterson, D., Gonzalez, J., Le, Q., Liang, C., Munguia, L.-M., Rothchild, D., So, D.,
Texier, M., & Dean, J. (2021). *Carbon Emissions and Large Neural Network Training*
(arXiv:2104.10350). arXiv. https://doi.org/10.48550/arXiv.2104.10350

Shilov, A. (2024, September 24). Intel launches Gaudi 3 accelerator for AI: Slower than
Nvidia's H100 AI GPU, but also cheaper. *Tom's Hardware*.
https://www.tomshardware.com/tech-industry/artificial-intelligence/intel-launches-gaudi-3-accelerator-for-ai-slower-than-h100-but-also-cheaper

Strubell, E., Ganesh, A., & McCallum, A. (2019). Energy and Policy Considerations for Deep
Learning in NLP. *Proceedings of the 57th Annual Meeting of the Association for
Computational Linguistics*, 3645 to 3650. https://doi.org/10.18653/v1/P19-1355

Wu, C.-J., Raghavendra, R., Gupta, U., Acun, B., Ardalani, N., Maeng, K., Chang, G.,
Behram, F. A., Huang, J., Bai, C., Gschwind, M., Gupta, A., Ott, M., Melnikov, A., Candido,
S., Brooks, D., Chauhan, G., Lee, B., Lee, H.-H. S., ... Hazelwood, K. (2022). *Sustainable
AI: Environmental Implications, Challenges and Opportunities* (arXiv:2111.00364). arXiv.
https://doi.org/10.48550/arXiv.2111.00364

---

Written by Dhwani Sanjay Kariya as coursework for *Emerging Artificial Intelligence
Technologies & Sustainability* (H9ETS), National College of Ireland.
