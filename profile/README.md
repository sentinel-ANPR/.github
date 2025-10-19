# Project Sentinel: Advanced ANPR & Vehicle Analysis Platform

**An intelligent, real-time system for Automatic Number Plate Recognition (ANPR), vehicle analytics, and automated traffic violation detection.**

---

## 1. Project Overview

As traffic management and road safety become increasingly critical, **Project Sentinel** offers a robust, scalable, and intelligent solution to enhance monitoring and enforcement. This is to be implemented at 11 locations around Kozhikode district as a pilot and has been commissioned by the office of the DCP Kozhikode district.

By leveraging state-of-the-art computer vision and a high-throughput, distributed architecture, Sentinel automates the entire process of identifying vehicles, analyzing their attributes, and flagging traffic violations. This system is designed to reduce manual intervention, increase operational efficiency, and provide powerful data-driven insights for public safety.

## 2. Core Features

* **High-Accuracy ANPR:** Detects and digitizes license plates in real-time, even in challenging low-light or adverse weather conditions.
* **Advanced Vehicle Analytics:** Goes beyond ANPR to identify key vehicle attributes, including **color**, **make (logo)**, and **model**.
* **Automated Violation Detection (Phase 2):** Automatically identifies and flags common infractions, including **no-helmet** and **triple-riding** on two-wheelers.
* **Centralized Dashboard:** A modern, web-based UI provides a centralized interface for live monitoring, reviewing violation events, and managing all collected data.
* **Massively Scalable:** The architecture is built to scale from a single camera to hundreds across multiple locations, ensuring high performance and reliability.

## 3. System Architecture: A Real-Time Microservice Approach

Sentinel is built on a modern, event-driven architecture designed for high throughput and fault tolerance. This design decouples tasks, allowing for independent scaling and processing.

### Architecture Diagram
   ┌────────────┐
   │  Camera    │
   │ Ingest     │
   │ (Process 1)│
   └─────┬──────┘
         │
         ▼
   ┌──────────────────┐
   │ Redis Stream     │
   │ "vehicle_jobs"   │
   └─────┬──────┬─────┘
         │      │      │
 ┌───────▼─┐  ┌─▼────────┐  ┌───────────┐  ┌─────────────┐
 │ OCR     │  │ Colour   │  │ Logo/Model│  │ Violation   │
 │ Worker  │  │ Worker   │  │ Worker    │  │ Worker      │
 │(Proc 2) │  │(Proc 3)  │  │(Proc 4)   │  │ (Proc 5)    │
 └───────┬─┘  └─┬────────┘  └───────┬───┘  └─────────┬───┘
         │      │                   │                │
         └──────┴───────────────────┴────────────────┘
                       results
                          │
                          ▼
                   ┌─────────────────────┐
                   │ Aggregator + API    │
                   │ (FastAPI Proc 6)    │
                   │  - listens to       │
                   │    "vehicle_results"│
                   │  - writes DB        │
                   │  - serves API       │
                   └─────────────────────┘

1.  **Camera Ingest:** A high-performance **Ingest Service** reads any number of RTSP streams. It uses YOLOv8 for real-time vehicle detection and tracking. When a unique vehicle is identified, it captures a high-resolution keyframe.
2.  **Event-Driven Bus (Redis):** This keyframe is published as a "job" to a **Redis Stream**. This acts as a central, persistent message bus, ensuring no jobs are lost and decoupling capture from analysis.
3.  **Specialized AI Workers:** A fleet of asynchronous, independent **AI Workers** subscribes to the job stream. Each worker performs a single, complex task concurrently:
    * **OCR Worker:** Runs on all vehicles to extract the license plate number.
    * **Violation Worker (Phase 2):** Scans motorcyclists for helmet and triple-riding violations.
    * **Color Worker:** Runs on cars to determine the vehicle's primary color.
    * **Logo/Model Worker:** Runs on cars to identify the manufacturer's logo.
4.  **Aggregator & API:** An **Aggregator Service** listens for results from all workers. It intelligently assembles the partial data (e.g., a `car` requires 3 results, a `motorcycle` requires 2) into a complete, final record. This record is then stored and made available instantly via a high-speed **FastAPI**.

This architecture ensures that a failure in one worker (e.g., `Logo Detection`) does not stop the processing of other data (like `OCR`), and the built-in retry logic guarantees that data is processed reliably.

## 4. Our Custom-Trained AI Models

The intelligence of Sentinel comes from a suite of custom-trained YOLOv8 models, each specialized for its task. We use a "One Corpus, Three Workflows" methodology, where a single base dataset is annotated in three different ways.

1.  **Model 1: 5-Class Vehicle Classification**
    * **Goal:** To accurately classify all vehicles and fix the common misclassification of auto-rickshaws.
    * **Classes:** `car`, `motorcycle`, `bus`, `truck`, `auto-rickshaw`

2.  **Model 2: Rider Violation Detection**
    * **Goal:** To detect all people on a motorcycle and classify their helmet status.
    * **Classes:** `person_with_helmet`, `person_without_helmet`
    * **Logic:** Violations are flagged if `person_without_helmet` is detected, or if the total count of persons is `> 2`.

3.  **Model 3: Vehicle Make/Model Identification**
    * **Goal:** A specialized small-object detector to identify manufacturer logos.
    * **Classes:** `tata`, `suzuki`, `hyundai`, `kia`, `mahindra`, `honda`, etc.

## 5. Technology Stack

* **AI & Computer Vision:** Python, YOLOv8 (PyTorch), OpenCV
* **Backend & API:** FastAPI
* **Message Bus & Caching:** Redis (Streams)
* **Database:** Sharded SQL/NoSQL for horizontal scaling
* **Frontend:** Modern JavaScript Framework (React/Vue/Svelte)

## 6. Project Roadmap

* **Phase 1: Core ANPR & Vehicle Analytics**
    * **Status:** In Progress
    * **Deliverables:** A high-accuracy ANPR system with full vehicle classification (5 classes) and advanced analytics (color, make/model).

* **Phase 2: Violation Detection Module**
    * **Status:** Pending
    * **Deliverables:** Integration of the `no-helmet` and `triple-riding` violation detection models into the main Sentinel platform.
