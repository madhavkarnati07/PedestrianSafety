# Pedestrian Safety: Edge-Based Object Detection and Tracking for Urban Safety Monitoring

## Abstract

This repository presents a research-oriented computer vision and edge AI system for pedestrian safety monitoring in mixed-traffic environments. The system is implemented using NVIDIA DeepStream, a real-time streaming analytics framework built for GPU-accelerated inference and multi-object tracking. The core objective is to detect and track vulnerable road users and other relevant scene objects in live camera streams and recorded video, enabling safety-focused decision support in pedestrian-heavy urban settings.

The project is designed around a practical deployment scenario: an edge device running object detection and tracking on incoming video, with visual overlays summarizing object counts and motion trajectories. In its current form, the implementation supports both USB camera input and prerecorded video files, making it suitable for experimental evaluation in both controlled and field-like conditions.

## Problem Statement

Pedestrian safety remains a critical concern in urban transportation systems, particularly in areas where insufficient situational awareness can lead to delayed reactions or unsafe interactions between vehicles, cyclists, and pedestrians. Traditional monitoring systems often rely on manually observed video feeds or offline post-processing, which limits responsiveness and scalability.

The central problem addressed by this project is the need for an efficient, edge-deployable system that can:

- detect pedestrians and other road participants in real time,
- maintain object identities over time despite motion and visual clutter,
- operate under varied lighting conditions, including daytime and nighttime scenes,
- provide practical deployment on embedded or edge hardware,
- support evaluation using either live streams or recorded footage.

This work sits at the intersection of smart mobility, computer vision, edge intelligence, and traffic safety. The goal is not only to recognize objects in frames but to generate robust temporal awareness of scene dynamics that can support proactive safety interventions.

## Research Motivation

As urban infrastructure continues to evolve toward intelligent transportation systems, there is growing interest in automated video analytics for safety-related applications. In real-world traffic scenes, a system must handle challenges including:

- occlusion between pedestrians and vehicles,
- variable scene illumination and shadows,
- fluctuating object scale and motion speeds,
- low-latency inference requirements,
- deployment constraints on embedded or edge devices.

This project addresses these challenges by leveraging DeepStream’s streaming pipeline architecture and NVIDIA GPU acceleration for efficient object detection and tracking. The repository demonstrates a proof-of-concept for pedestrian safety monitoring using a production-ready inference stack rather than a purely offline algorithmic prototype.

## System Overview and Design

The system follows a modular streaming pipeline architecture typical of modern edge AI video analytics systems.

### High-Level Architecture

The processing pipeline consists of the following stages:

1. Video acquisition
   - USB camera input via V4L2 (`v4l2src`)
   - prerecorded video file input via `filesrc`

2. Video decoding and format conversion
   - hardware decode (`nvv4l2decoder` or equivalent GStreamer decode path)
   - conversion to a format suitable for inference and display

3. Stream batching and synchronization
   - `nvstreammux` groups incoming frames into processing batches

4. Primary inference
   - `nvinfer` runs the primary object detection model
   - object classes include vehicle, bicycle, person, and road sign

5. Tracking
   - `nvtracker` maintains object identities across frames
   - this is essential for interpreting movement continuity in traffic scenes

6. Secondary inference and refinement
   - optional additional inference stages can be configured for more specialized detection logic

7. Display and visualization
   - `nvdsosd` overlays bounding boxes, labels, and metadata
   - output is displayed through an EGL sink or hardware display path

8. Metadata inspection
   - object counts and frame-level analytical summaries are logged via metadata probes

### Core Design Principles

The design prioritizes the following:

- real-time processing on edge hardware,
- compatibility with NVIDIA DeepStream-based deployment,
- modularity through configuration files,
- visual explainability using annotated frames,
- support for both live and recorded modes.

## Methodology

### Data Sources

The project uses two common deployment modalities:

- recorded traffic or pedestrian video sequences stored locally,
- live USB camera streams captured from an edge device.

The repository includes example video outputs in the `Results/` directory, such as:

- `DayNear.mp4`
- `DaySide.mp4`
- `DaytimeLong.mp4`
- `Night_Near.avi`
- `night_long.avi`
- `night_near_car.avi`

These clips provide qualitative evidence for system behavior across different circumstances, including daylight and nighttime conditions.

### Detection Pipeline

The project configures a primary detector using the DeepStream pipeline configuration pattern. The inference configuration defines the following object categories:

- vehicle
- bicycle
- person
- road sign

The primary detection stage is followed by tracking logic, enabling the system to estimate motion continuity and maintain object IDs across frames. The configuration and tracker setup indicate a focus on robust, real-time perception rather than static frame-by-frame classification alone.

### Tracking Strategy

The tracker is configured through `dstest2_tracker_config.txt` and uses the NvDCF tracker configuration, which is appropriate for multi-object tracking in video streams. NvDCF-based tracking is useful when objects may exhibit partial occlusion, nonlinear motion, or frequent re-identification challenges. The tracker is configured for a fixed image size and GPU execution, which is consistent with production-grade edge inference pipelines.

### Metadata and Output Interpretation

The code adds a probe on the OSD sink buffer to inspect metadata generated during inference. This metadata includes:

- frame number,
- number of detected objects,
- per-class object counts,
- information about tracked objects and their identities.

This design makes the system useful not only for visualization but also for empirical analysis of object counts and temporal behavior. While the current repository does not report formal numerical performance benchmarking, the code structure is designed to support such evaluation.

## Testing and Evaluation Methodology

### Test Scenarios

The repository is structured to support evaluation in multiple operational contexts:

- video-file processing for repeatable benchmarking,
- live camera processing for real-time deployment assessment,
- daylight and nighttime video sequences for scene-variability analysis,
- mixed pedestrian and vehicle conditions for safety-relevant use cases.

### Qualitative Testing Procedure

The project appears to evaluate performance through visual inspection of annotated output videos. The testing flow is aligned with the following protocol:

1. Acquire input stream (camera or file)
2. Run the DeepStream pipeline with detection and tracking enabled
3. Validate that objects are recognized and tracked across frames
4. Inspect the output visualizations for detection stability
5. Record qualitative observations for different lighting and motion conditions

The clips in `Results/` serve as evidence of this workflow and demonstrate the system’s ability to generate annotated output under varying conditions.

### Practical Evaluation Dimensions

Although the repository does not include a formal benchmark report, the implementation is designed to support evaluation on these dimensions:

- detection continuity across frames,
- class-specific object visibility,
- robustness to scene changes,
- performance under low-light scenes,
- latency and throughput under edge deployment conditions,
- reliability of object tracking over time.

## Results and Observations

The repository includes video demonstrations showing that the system can process both daytime and nighttime traffic scenes and overlay detection output on the original stream. These results are particularly relevant to pedestrian safety applications, where the ability to maintain situational awareness in changing environments is critical.

The results suggest the following qualitative outcomes:

- the pipeline successfully detects multiple object classes in video sequences,
- annotation overlays enhance interpretability for human observers,
- the tracker allows temporal continuity for detected objects,
- daytime and nighttime scenes are both represented in the repository outputs,
- the system demonstrates feasibility for edge-based urban monitoring.

It is important to note that the available repository artifacts represent demonstration-level evidence rather than a full experimental benchmark. The project is best viewed as a functional prototype and a research foundation for further quantitative evaluation.

## Repository Structure

```text
PedestrianSafety/
├── Results/
│   ├── DayNear.mp4
│   ├── DaySide.mp4
│   ├── DaytimeLong.mp4
│   ├── Night_Near.avi
│   ├── night_long.avi
│   ├── night_near_car.avi
│   └── live demo.mp4
├── project/
│   ├── README
│   ├── deepstream-usbcam/
│   │   └── deepstream_test_1_usb.py
│   └── deepstream_detect_track/
│       ├── deepstream_test_2.py
│       ├── dstest2_pgie_config.txt
│       ├── dstest2_sgie1_config.txt
│       ├── dstest2_sgie2_config.txt
│       ├── dstest2_tracker_config.txt
│       └── config_tracker_NvDCF_perf.yml
└── installation/
```

## How to Run

### Run on a recorded video

```bash
cd project/deepstream_detect_track
python3 deepstream_test_2.py <video_file_name>
```

### Run on a live USB camera

```bash
cd project/deepstream-usbcam
python3 deepstream_test_1_usb.py <device_path>
```

Example:

```bash
python3 deepstream_test_1_usb.py /dev/video0
```

## Hardware and Software Context

This project is designed for NVIDIA DeepStream-enabled environments and is most appropriate for systems with:

- NVIDIA GPU acceleration,
- DeepStream SDK installed,
- GStreamer support,
- Python 3 runtime,
- compatible camera or video input sources.

The implementation uses DeepStream’s Python bindings (`pyds`) and GStreamer pipelines, making it a practical demonstration of edge AI video analytics in a transportation safety context.

## Limitations and Future Work

This repository is best interpreted as an applied research prototype. Several areas remain open for improvement:

- quantitative benchmark evaluation,
- accuracy analysis under diverse weather and illumination conditions,
- pedestrian-specific tracking robustness in crowded scenes,
- integration with alerting or risk-estimation modules,
- deployment on embedded edge devices with power constraints,
- fusion with other sensing modalities such as LiDAR or radar.

Future iterations could extend this work toward a full driver-assistance or smart-city monitoring framework that translates detected motion patterns into actionable safety metrics.

## Conclusion

This project addresses a significant societal and engineering problem: the need for real-time perception systems that can help prevent pedestrian-related hazards in urban traffic environments. By combining DeepStream-based object detection, multi-object tracking, and live video processing, the repository demonstrates a practical edge AI architecture for pedestrian safety analytics.

The current implementation provides a strong foundation for continued development in safety-critical computer vision and intelligent transportation systems. Its research value lies in taking a production-grade streaming pipeline and adapting it to the problem of pedestrian monitoring, with a clear path toward more rigorous benchmarking and deployment-oriented optimization.

## License

The project includes NVIDIA DeepStream sample code and configuration patterns, consistent with the repository’s original source structure and licensing references.

