# FOREST-ACOUSTIC-LOCATOR
Edge-AI forest acoustic monitoring: on-device sound classification, BLE event sharing, multi-node TDOA localization, and predictive maintenance for remote equipment.
Forest Acoustic Locator for Forest Monitoring (FAL)
Project status: early prototype scaffold. The hardware pipeline, trained model, BLE link, and field localization accuracy have not yet been validated. This repository documents the design and provides a first gateway-side TDOA solver for synthetic/test measurements.

(FAL) is a proposed low-power acoustic monitoring network for forest conservation and remote infrastructure. Sensor nodes run an Edge AI sound classifier locally, then share event metadata over Bluetooth Low Energy (BLE). When three or more spatially separated microphones capture the same event, a gateway estimates the source position using audio-based time-difference-of-arrival (TDOA) measurements and known sensor coordinates.

What it is intended to do
- Classify selected sound categories on the sensor node: background/unknown, selected wildlife calls, gunshot-like transients, chainsaw/vehicle sounds, and normal/abnormal pump or generator sounds.
- Run inference locally to avoid continuously streaming all audio to a cloud service.
- Send event class, confidence, node ID, timestamp, node health, and selected audio snippets over BLE.
- Align recordings from multiple nodes and estimate TDOA using cross-correlation/GCC-PHAT at a gateway.
- Estimate a two-dimensional source location using sensor coordinates surveyed before deployment.
- Track persistent acoustic deviations in machinery and recommend inspection rather than claiming a fault has been diagnosed.
- Evaluate classification quality, false alarms, localization error, latency, packet loss, and energy use with repeatable tests.
Hardware platform
The contest-oriented development target is the Nordic nRF54LM20 DK. For the full three-node localization experiment, the intended configuration is three microphone-equipped sensor stations. Begin with one DK to bring up microphone capture, model inference, and BLE; add the other stations before claiming multi-node localization.

The nRF54LM20 DK supports nRF54LM20A and nRF54LM20B targets. Nordic documents Axon NPU / Edge AI Add-on support for the nRF54LM20B target; confirm the board/SoC target and installed SDK before relying on NPU acceleration. A compatible PDM digital microphone is planned, but microphone wiring, pin assignment, and I/O voltage must be checked against the current DK hardware documentation before connecting hardware.

See [`hardware/BOM.csv`](hardware/BOM.csv) for the starter bill of materials.
System architecture
1. Audio capture: PDM microphone → PDM peripheral/EasyDMA → PCM frame buffer.
2. Edge AI inference: preprocess audio (for example, log-mel features if required by the chosen model) and run the quantized classifier on the node. Include an unknown/low-confidence outcome.
3. Event buffer: maintain a rolling pre-trigger audio buffer and preserve a short clip around a detected event.
4. BLE event transport: send node ID, event class, confidence, local timestamp/sequence number, health status, and optional short audio data to the gateway.
5. Gateway alignment/localization: retrieve overlapping recordings, estimate relative delays with cross-correlation/GCC-PHAT, then solve the TDOA equations from surveyed node positions.
6. Map and alerts: display an estimated location and uncertainty/residual quality; do not present a result as precise until it is validated against known source positions.
7. Predictive maintenance: establish a healthy baseline for a pump/generator and watch for sustained changes in anomaly scores or acoustic features.
See [`docs/architecture.md`](docs/architecture.md), [`docs/localization.md`](docs/localization.md), and [`docs/predictive-maintenance.md`](docs/predictive-maintenance.md).
Important localization facts
- Three non-collinear microphones are a practical minimum for 2D TDOA, when relative timing is sufficiently accurate and the geometry is suitable. Three nodes do not guarantee a unique or precise result in every geometry; four or more nodes provide redundancy and are recommended for field tests.
- For 3D source localization, use at least four suitably placed microphones, usually with non-coplanar geometry, and expect more demanding timing and geometry requirements.
- BLE packet arrival time is not the acoustic arrival time. BLE transports metadata/recordings; estimate sound delays from the recorded audio or rigorously calibrated acquisition timestamps.
- Independent audio sample clocks, microphone response delays, echoes, wind, terrain, and low signal-to-noise ratio can bias TDOA estimates.
- A sound classifier predicts a label. It does not prove that a gunshot occurred or that a call belongs to a particular species. Measure false positives and retain an unknown class.
The included Python script estimates a 2D source position from sensor coordinates and measured arrival times. It does not extract delays from audio yet, and the included example is synthetic.

Input fields use local Cartesian coordinates in metres and arrival times in seconds. The script assumes a uniform speed of sound and a common timing reference. Real recordings require timestamp/sample-rate calibration and an audio cross-correlation stage; see [`gateway/README.md`](gateway/README.md).
Planned milestones
- [ ] Confirm DK revision/target and establish a supported nRF Connect SDK toolchain.
- [ ] Connect and validate a compatible PDM microphone; verify PCM sample rate and clipping/noise.
- [ ] Build a labelled dataset with permitted audio, documented sources, and separate train/validation/test sets.
- [ ] Train and evaluate a small sound classifier; quantize and deploy it using a supported Nordic Edge AI workflow.
- [ ] Report labels, confidence, event timestamps, and node health over BLE.
- [ ] Preserve overlapping audio clips and test transfer/throughput limits.
- [ ] Implement GCC-PHAT or another validated audio delay estimator.
- [ ] Test TDOA with three or more surveyed microphone positions and known source locations.
- [ ] Add confidence checks, residual thresholds, and map display.
- [ ] Collect healthy/abnormal machinery audio and evaluate anomaly warnings for predictive maintenance.
- [ ] Measure precision/recall, false alarms, location error, latency, packet loss, and average current.
- [ ] Document field tests, limitations, and safety/privacy considerations.
Ethical, privacy, and safety notes
Use the system for conservation, biodiversity research, and authorized infrastructure monitoring. Handle location data for vulnerable species carefully and restrict access when disclosing locations could increase poaching risk. Follow local laws and obtain permission before audio monitoring. Do not treat model predictions as definitive evidence or as a substitute for human review.
References
- Nordic nRF54LM20 DK — get started
- Nordic nRF54LM20 DK — hardware files
- Nordic Edge AI
- Nordic Axon NPU
- nRF54LM20 PDM peripheral documentation
- Sustain the System contest
License
The repository's documentation and starter code are released under the MIT License. Third-party audio recordings, trained model weights, and external assets are not automatically covered by this code license; follow the terms of each source and document dataset provenance.
