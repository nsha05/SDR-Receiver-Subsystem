# SDR Receiver Subsystem (ECE 295)

During the winter semester of 2025, I led a team of three students to design, build, and test a **Receiver (RX) Subsystem** of a software-defined radio (SDR) for the design course ECE295.

![Final PCB](images/final_pcb.jpg)

## My role as Team Lead

The course had 10 major deadlines spread over the term (reports, technical milestones, presentations, PCB submissions), so a lot of my job was figuring out who was doing what each week and making sure we didn't fall behind. This also included taking on extra responsibility to ensure all team members' contributions were up to standard, and reviewing everyone's work before each submission. I was also the main spokesperson who was in charge of external communications, including contact with the Teaching Assistants for technical questions and demonstrations, as well as the Communications Instructors for written submissions and presentations. 

[Initial Design Spec Proposal, Feb 2025](docs/DesignProposal_Team5.pdf)

[Design Proposal Presentation, Feb 2025](docs/OP1_Proposal.pdf)

[Final Design Review, Apr 2025](docs/Final_Design_Review.pdf)

---

## What the subsystem does

The receiver takes the incoming radio frequency (RF) signal from the receive (RX) switch, takes only our desired frequency band, mixes it down to baseband as two signals 90° apart (In-Phase and Quadrature), and amplifies them for the demodulator (Subsystem B). From then onwards, the signal undergoes further processing through the various other subsystems. The local oscillator signals that drive the mixer come from Subsystem C.

```
RX_SIG → Enable Switch → Bandpass Filter → Voltage Clipper → Quadrature Mixer → Low-pass Filter → I/Q Amplifier → RX_I, RX_Q
                                                                  ↑
                                                       LO_0° / LO_90° (Subsystem C)
```
---

## Planning

The major milestone layout of the term was as follows:

| Date | Milestone |
|---|---|
| Jan 24 | Written Document 0 |
| Feb 10 | Technical Milestone 0, Written Document 1 |
| Feb 24 | Technical Milestone M1, Oral Presentation 1 (design proposal) |
| Mar 10 | Technical Milestone M2 |
| Mar 16 | Final PCB submission, parts order |
| Apr 1 | Technical Milestone M3 |
| Apr 2–29 |PCB assembly and testing|
| Apr 30 |Final Design Review |
---

## Initial proposed design

Everything within our design had to meet the course's Interface Control Document (ICD), with a tight timeline and a strict budget. That pushed us toward designs that were simple, reliable, and cheap, rather than too overcomplicated and likely to induce error. 

![Initial Design Schematic](images/proposed_schematic.png)

## First major obstacle

For every major stage of the receiver, we went with whatever was cheapest and easiest to build that still met the specifications. We decided to create a 3 stage bandpass filter because more stages would have added to both cost and board space, with little performance improvement. We also used simple diode and op-amp circuits for the limiter and amplifier. 

That's also why I decided to try and build my own quadrature mixer instead of buying an integrated circuit (IC) like most other teams. However, after many proposed quad-mixer designs all not performing as expected, I wasn't sure if the extra effort was yielding worthwhile results. Ultimately it turned out to be a lot harder than I expected, and I ended up buying one anyways to make sure we didn't fall behind on our time constraints :(

I replaced my non-functional DIY mixer with a cheap but reliable quadrature mixer IC (ADE-1+)

![Mixer design change: discrete transformer and MOSFET mixer replaced by a Mini-Circuits module](images/design_change_mixer.png)

## Final design

After a numerous simulations and more minor tweaks, we had our final design:

![Final schematic](images/final_schematic.png)

I created the schematic and PCB layout in Altium Designer and submitted the board for fabrication in mid-March. This ensured that we stayed on track with our original timeline, so my team would have enough time to assemble and debug the board before our final demonstration and design review.

![PCB layout in Altium](images/pcb_layout.png)

## PCB Assembly

We assembled the board in the Myhal Centre Fabrication Facility (MYFab), using a soldering iron for most components, as well as soldering paste and a heat gun for the few surface-mounted components. After some basic continuity checks to ensure everything was connected properly, we then mounted our PCB on the radio motherboard.

<img src="images/board_on_motherboard.jpg" width="600" alt="RX subsystem on the radio motherboard">

## Testing procedure

We characterized each stage on the finished board with an oscilloscope, waveform generator, and power supply.

I also automated python scripts to sweep across frequencies to create the bode plots for our final ICD checks.

<img src="images/oscilloscope.jpg" width="500" alt="Oscilloscope during testing">

| Bandpass filter | I/Q phase difference | Low-pass filter |
|---|---|---|
| ![BPF frequency response](images/test_bpf.png) | ![I/Q phase vs frequency](images/test_mixer.png) | ![LPF frequency response](images/test_lpf.png) |

Ultimately, our board was mostly functional! The signal behaved generally as expected as it moves through the different components of our receiver. However, it wasn't perfect:

| Component | Requirement | Measured | Error | Pass? |
|---|---|---|---|---|
| Enable switch | 3.3 V high | 3.3 V | 0% | ✅ |
| Bandpass filter | 8–16 MHz | ~8–14.5 MHz | ~0% (low edge), ~9.4% (high edge) | ❌ |
| Voltage clipper | 0.7 V | 0.7 V | ~0% | ✅ |
| Quadrature mixer | −90° I/Q phase shift | ~−90° | ~0% | ✅ |
| Low-pass filter | 96 kHz cutoff | ~20 kHz | ~79.2% | ❌ |
| I/Q amplifier | >30 dB gain | >30 dB | 0% | ✅ |

While every stage worked for the most part, our bandpass filter and low-pass filter had some issues meeting the desired cutoff frequency. However, both of these filters had previously been shown to be working in our LTSpice simulations. After spending so much time on the board, it was really important for me to understand why the final measurements weren't matching my simulations.

One thing I hadn't considered during my simulations was the impact of the different modules on each other. I had tested each part of the receiver (bandpass filter, voltage clipper, low-pass filter, amplifier) separately. However, on the actual board, it's possible that the low-pass filter cascaded with the high-pass filter part of our IQ Amplifier design, and dragged the cutoff down. I  hadn't caught it earlier because we'd been testing each section on its own. The fix is the just recalculate the component values for the cascaded circuit so the cutoff moves back up to 96 kHz.

The bandpass filter upper cutoff was also slightly off, but not by a large amount. One detail I noticed after reviewing for potential causes of error was the tolerances of our components. A few of our parts, mainly the inductors, had ±20% tolerance. It's possible this tolerance was was enough to pull the upper cutoff down due to inaccurate values. Lower tolerance components should be able to fix this issue.

**Major Takeaways**
- Test parts of neighbouring stages together to identify how they might impact each other
- Build component tolerance into the simulation from the start, or spend a bit more of the budget on better parts

## Conclusion

Looking back, this was one of the most hands-on and rewarding projects I've done so far. It was the first time I took an idea from my brain to a physical board I could actually build and measure. Seeing the signal actually make it through our receiver and meet almost all the ICD requirements really made me feel like my efforts were worth it (even if it wasn't perfect)

The part I unexpectedly learned from the most was managing the team itself. Three people with different schedules, opinions, and work ethics meant we didn't always agree on the same thing. As team lead I constantly had to step in and make a decision, even when that meant not everyone was happy with the outcome. It taught me a lot about leadership in general, and how working on a team almost never goes exactly as planned. No matter how many contingency plans you lay out in the beginning, some unexpected situation is going to pop up eventually, and knowing how to deal with it on the fly is a really important skill to have.

Overall, working on this project was an amazing experience, and I learned a lot. I'm really grateful to my team, my TA, CI, as well as professor Sean Hum for putting together such an engaging design course :)
