# A.U.R.A.-Autonomous-Unitree-Response-Agent-
# G1-EDU Local Voice Control Interface

**Status:** In Entwicklung (Pre-Alpha)  
**Plattform:** Unitree G1 EDU (Development Unit)  
**Middleware:** ROS2 Humble

## 1. Projektübersicht
Dieses Projekt zielt darauf ab, den Unitree G1 EDU Humoid-Roboter vollständig **lokal und offline** per Sprache zu steuern. 
Standardmäßig setzt Unitree (ab Firmware 1.3.0) auf eine Cloud-Anbindung via GPT. Wir ersetzen diesen Pfad durch eine datenschutzfreundliche On-Device-Pipeline, die direkt auf der **Development Computing Unit (DU)** des Roboters (oder einem verbundenen Edge-Rechner) läuft.

Das Ziel: **Sprache → Motion Mapping** (Umwandlung natürlichsprachlicher Befehle in ROS2-Topics).

## 2. Systemarchitektur

Die Pipeline ist modular aufgebaut, um Latenzen zu minimieren und Komponenten austauschbar zu machen:

1.  **Input:** Audio-Stream (Mikrofon-Array oder USB-Mic).
2.  **Wake Word Detection:** Aktivierung durch Keyword (z.B. "Hey Unitree") via **Porcupine** (geringe Latenz).
3.  **ASR (Speech-to-Text):** Lokale Transkription mittels **Whisper** (cpp-Implementierung).
4.  **Intent Recognition (LLM):** Ein quantisiertes SLM (Small Language Model) extrahiert die Absicht.
    * *Modelle:* Llama 3 8B Q4, Mistral 7B oder Phi-3 (3.8B).
    * *Output:* JSON oder definierte Funktionsaufrufe.
5.  **ROS2 Bridge:** Mapping der Intents auf High Motion Development (HMD) Commands.
6.  **Execution:** Der `g1_controller` führt die Bewegung aus.
7.  **Feedback (Optional):** Audio-Antwort via **Piper TTS**.

## 3. Tech Stack

| Bereich | Technologie | Details |
| :--- | :--- | :--- |
| **OS** | Ubuntu 22.04 | Standard für ROS2 Humble |
| **Middleware** | **ROS2 Humble** | Kommunikation & Nodes |
| **Wake Word** | **Porcupine** | Effizient, Offline, Custom Keywords |
| **STT / ASR** | **Whisper.cpp** | OpenAI Whisper (optimiert für CPU/Edge) |
| **LLM Inference** | **Llama.cpp / Ollama** | GGUF Quantisierung (Q4_K_M) |
| **TTS** | **Piper** | Schnelle, neurale Sprachsynthese |
| **Hardware SDK** | **Unitree SDK2** | RPC Schnittstelle zum Roboter |

## 4. Hardware Setup & Verbindung

### Voraussetzungen
* **Unitree G1 EDU** (Zugriff auf die interne Development Unit ist zwingend).
* **Entwickler-PC** (z.B. Laptop mit Ubuntu/WSL für Cross-Development).
* **Verkabelung:** Ethernet-Kabel + USB-C Adapter.

### Verbindung herstellen (Quick Development)
Siehe [Unitree Developer Guide](https://support.unitree.com/home/en/G1_developer/quick_development).

1.  Ethernet-Verbindung zwischen PC und G1 herstellen.
2.  IP-Adresse konfigurieren (Subnetz beachten).
3.  SSH-Zugriff auf die DU:
    ```bash
    ssh unitree@<ROBOT_IP>
    ```

## 5. Roadmap

### Phase I: Infrastruktur & Basic Motion
- [ ] SSH-Verbindung zur DU stabilisieren.
- [ ] High Motion Development (HMD) Commands via Terminal/Skript testen.
- [ ] Mikrofon-Hardware evaluieren (intern vs. extern).

### Phase II: Local Audio Pipeline
- [ ] Porcupine Wake Word integrieren ("Lauf nicht dauerhaft").
- [ ] Whisper lokal deployen und Performance testen.
- [ ] Text-to-Speech (Piper) Output testen.

### Phase III: Intelligence Integration
- [ ] LLM Prompt-Engineering: System-Prompt für JSON-Output erstellen.
- [ ] ROS2 Node: `Subscription (Text)` -> `Publication (Motion)`.
- [ ] Integration komplexer Befehle ("Nimm die Flasche...").

## 6. Technische Herausforderungen & Constraints

Da das System lokal laufen soll, müssen folgende Punkte kritisch beachtet werden:

* **Rechenleistung der DU:** Die internen Ressourcen sind begrenzt. Ein paralleler Betrieb von Whisper, LLM und ROS2 Controller kann zu Überlastung führen.
    * *Lösung:* Nutzung stark quantisierter Modelle (Phi-3, TinyLlama) oder Offloading auf einen dedizierten Jetson/NUC Rucksack-PC.
* **Latenz (Latency):** Die Kette ASR -> LLM -> Motion dauert.
    * *Lösung:* Implementierung von **Regex-basierten Not-Aus-Befehlen**, die das LLM umgehen ("STOPP"), um Sicherheit zu garantieren.
* **Akustik:** Eigengeräusche des Roboters (Lüfter, Motoren) stören das Mikrofon.
    * *Lösung:* Testen von direktionalen USB-Mikrofonen oder Software-Filtern (Noise Suppression).
* **Halluzinationen:** LLMs können ungültige Befehle generieren.
    * *Lösung:* Strict Grammar Constraints (grammar files) im LLM nutzen, um nur validen Syntax zuzulassen.
## 7. Referenzen
* [Unitree G1 Developer Guide](https://support.unitree.com/home/en/G1_developer/rpc_routine)
* [ROS2 Humble Documentation](https://docs.ros.org/en/humble/)
