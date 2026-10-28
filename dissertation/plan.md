# Project Plan Portfolio

## Project Title

**Designing and Evaluating a Multimodal Robotic Facilitator for Supporting Face-to-Face Conversation Between Unacquainted People**

---

## 1. Aims, Research Question and Objectives

### 1.1 Project Aim

The aim of this project is to design, implement, and evaluate a multimodal robotic conversation facilitator that supports face-to-face interaction between unacquainted people. The system will use speech, conversation history, and limited visual context interpreted through existing multimodal large-model APIs to help the robot provide light, safe, and non-intrusive conversational support.

The project does not aim to train a new multimodal model or claim a completely new robotic architecture. Instead, it focuses on integrating available AI capabilities into a working human-robot interaction framework and evaluating whether the resulting system can improve human-human conversation in a realistic setting.

The robot will be designed as a facilitator rather than as a normal chatbot or a third conversation partner. It should avoid speaking continuously. Instead, it should provide structured prompts or light conversational support when the interaction appears to need help, while remaining unobtrusive when participants are already conversing naturally.

### 1.2 Research Question

**How can a multimodal robotic facilitator use speech, dialogue history, and visual context to support face-to-face conversation between unacquainted people, and what effect does it have on conversation flow, silence, and perceived interaction quality?**

### 1.3 Sub-Questions

1. Can a working multimodal robotic facilitation framework be implemented within the MSc project timeframe using existing large-model APIs and available robotic hardware?

2. Does the presence of the robotic facilitator reduce conversational silence and support more sustained interaction compared with a no-robot condition?

3. How do participants perceive the robot in terms of helpfulness, naturalness, friendliness, comfort, intrusiveness, and overall conversation experience?

4. What practical and ethical limitations emerge when a robot uses speech and visual context to support human-human conversation?

### 1.4 Objectives

1. **To review relevant literature** on social robots, human-human-robot interaction, ice-breaking technologies, robot-mediated social facilitation, multimodal interaction, and evaluation methods for human-human conversation.

2. **To design a feasible multimodal robotic facilitation framework** that combines speech input, conversation history, limited visual context interpreted through existing multimodal large-model APIs, response generation, and embodied robot output.

3. **To implement a working prototype** that can first run as a local simulation and then be integrated with the physical robot, using existing APIs rather than training a new multimodal model.

4. **To investigate how limited visual context can support facilitation**, for example by helping the system identify broad interaction-relevant cues such as whether participants appear oriented toward the conversation, visibly disengaged, silent, or in need of a light prompt.

5. **To conduct pilot testing** to assess system reliability, multimodal input quality, timing, prompt appropriateness, robot output quality, and whether the system behaves in a socially acceptable and non-intrusive way.

6. **To conduct a small-scale user evaluation** comparing conversations with and without the robotic facilitator, using objective measures such as silence duration, speaking turns, and conversation duration, alongside subjective measures such as comfort, awkwardness, naturalness, helpfulness, and perceived intrusiveness.

7. **To analyse the results and limitations** in order to identify when multimodal robotic facilitation is useful, when it may be unnecessary or intrusive, and what improvements would be needed for future deployment.

---

## 2. Summary of Proposed Research and Analysis Methodology

### 2.1 Research Design

This project will use a design-and-evaluation methodology. The first part of the project will focus on building a working multimodal robotic facilitation framework. The second part will evaluate the system through a controlled small-scale human participant study.

The project will be organised into two main phases. The first phase will focus on technical development, integration, and pilot testing. The second phase will focus on formal user evaluation after the system has reached a stable version. This avoids repeatedly changing the system during participant evaluation and helps ensure that the collected data reflects a consistent experimental condition.

The central research contribution is not the development of a new foundation model. Instead, the contribution is the design of a working multimodal robotic facilitator and the empirical evaluation of its effect on human-human conversation.

### 2.2 Proposed System Framework

The system will follow a lightweight multimodal facilitation pipeline:

```text
Human speech and visual context
        ↓
Multimodal context interpretation through existing API
        ↓
Conversation history and interaction state tracking
        ↓
Facilitation decision
        ↓
Context-aware prompt generation
        ↓
Robot speech / gesture / optional tablet output
        ↓
Interaction logging
```

The first prototype will be implemented as a local text-based or terminal-based simulation where two users can enter messages and the system decides whether a robotic facilitator should respond. This reduces early hardware risk and allows the basic conversation-control loop to be tested before robot integration.

The robot-integrated version will use speech input or transcripts, simple visual input where feasible, response generation, robot speech output, and simple embodied behaviour. A tablet or lightweight web interface may be used as an optional visual display for safe topic cards, images, or prompts, but the core project will not depend on a complex tablet application.

### 2.3 Multimodal Context Understanding

The system will use existing multimodal large-model APIs to interpret selected visual inputs from the interaction environment. The visual input will not be used to identify participants, infer sensitive personal attributes, or make complex psychological judgements. Instead, it will be used only to extract simple interaction-relevant cues, such as whether participants appear oriented toward each other, whether one participant is visibly disengaged, whether people are looking away for a prolonged period, or whether there is a long pause combined with low visible engagement.

This project will not train a new vision-language model. The contribution lies in integrating multimodal perception into a robotic conversation facilitation framework and evaluating whether the resulting system supports human-human interaction.

### 2.4 Core System Components

| Component | Planned role |
|---|---|
| Speech / transcript input | Capture or receive utterances from two participants. In early prototypes this may be typed input; in the robot version it may use speech transcription or semi-structured operation depending on hardware reliability. |
| Visual context input | Use limited visual input interpreted through an existing multimodal API to extract broad interaction cues. |
| Conversation history | Store recent utterances, speaker turns, and robot interventions. |
| Interaction state tracking | Identify simple states such as active conversation, extended silence, repeated short replies, topic exhaustion, or unbalanced participation. |
| Facilitation decision | Decide whether the robot should remain silent or provide a light prompt. This will use simple rules and prompt templates rather than an advanced proactive reasoning model. |
| Response generation | Generate safe, short, and context-aware prompts, such as follow-up questions, neutral topic suggestions, or invitations for the quieter participant to contribute. |
| Embodied output | Use robot speech and simple gestures. Optional tablet output may be added if it improves clarity or engagement. |
| Logging | Record timestamps, speaking turns, robot interventions, silence periods, condition labels, and questionnaire data for analysis. |

### 2.5 Baseline and Evaluation Conditions

The formal evaluation will focus on a small-scale controlled comparison between:

```text
Condition A: conversation without robot
Condition B: conversation with multimodal robotic facilitator
```

A scripted-prompt robot condition may be used as an optional pilot or extension if time, recruitment, and system stability allow. However, the core evaluation will not depend on this additional condition.

The no-robot condition is important because the main research question concerns whether the robotic facilitator affects human-human conversation. The robot condition will test whether the working system can support interaction in ways that are measurable and perceptible to participants.

### 2.6 Objective Measures

The project will collect objective interaction measures such as:

- total conversation duration or active speaking time;
- total silence duration;
- longest silence period;
- number of speaking turns;
- approximate balance of participation between speakers;
- number of robot interventions;
- whether participants continue speaking after a robot intervention.

These measures will provide evidence about whether the robot affects the structure and flow of the conversation.

### 2.7 Subjective Measures

After each session, participants will complete a short questionnaire. The questionnaire will use Likert-scale items and may include constructs such as:

- perceived awkwardness;
- ease of starting or continuing conversation;
- comfort;
- naturalness of the robot’s intervention;
- helpfulness;
- friendliness;
- fun or lightness of the atmosphere;
- perceived intrusiveness;
- willingness to use such a system again.

Optional short post-study interviews may be used to collect qualitative feedback, especially about when the robot helped, when it interrupted too much, or when its behaviour felt unnatural.

### 2.8 Analysis Method

The analysis will combine quantitative and qualitative interpretation.

For objective measures, the project will compare the no-robot and robot conditions using descriptive statistics such as means, medians, ranges, and visual summaries. If the sample size is sufficient, simple statistical comparisons may be used.

For questionnaire results, Likert-scale responses will be summarised by condition. Qualitative comments and observations will be grouped into recurring themes, such as helpful prompting, awkward intervention, improved comfort, novelty effect, or perceived intrusiveness.

Because this is an MSc project with limited time and participant access, the evaluation will be designed as a small but controlled study rather than a large-scale user study. The goal is to provide credible evidence and useful design insights, not to claim broad generalisation.

---

## 3. Gantt-Style Timeline

**Assumption:** Week 1 starts in early June. If the official project start date or dissertation deadline differs, the week numbers should be shifted while preserving the relative duration and ordering of tasks.

Legend:

```text
● = active work
◆ = milestone
△ = optional / stretch task
```

| Task | W1 | W2 | W3 | W4 | W5 | W6 | W7 | W8 | W9 | W10 | W11 | W12 | W13 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Literature review and research framing | ● | ● | ● | ● | ● |  |  |  |  |  |  |  |  |
| Ethics preparation and ERGO materials | ● | ● | ● | ◆ |  |  |  |  |  |  |  |  |  |
| System architecture design | ● | ● | ◆ |  |  |  |  |  |  |  |  |  |  |
| Local simulation / MVP conversation loop | ● | ● | ● | ◆ |  |  |  |  |  |  |  |  |  |
| Multimodal API integration and testing |  | ● | ● | ● | ● | ◆ |  |  |  |  |  |  |  |
| Robot integration: speech, gestures, logging |  | ● | ● | ● | ● | ◆ |  |  |  |  |  |  |  |
| Optional tablet / web interface |  |  | △ | △ | △ |  |  |  |  |  |  |  |  |
| Technical testing and prompt refinement |  |  | ● | ● | ● | ◆ |  |  |  |  |  |  |  |
| Pilot testing with friends or approved volunteers |  |  |  |  | ● | ● | ◆ |  |  |  |  |  |  |
| Finalise evaluation protocol and study materials |  |  | ● | ● | ● | ● | ◆ |  |  |  |  |  |  |
| Formal participant evaluation, minimum two weeks |  |  |  |  |  |  | ● | ● | ◆ |  |  |  |  |
| Quantitative result cleaning and analysis |  |  |  |  |  |  |  |  | ● | ● | ◆ |  |  |
| Qualitative feedback analysis |  |  |  |  |  |  |  |  | ● | ● | ◆ |  |  |
| Dissertation writing and figure/table preparation | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ◆ |
| Final editing, limitation analysis, viva preparation |  |  |  |  |  |  |  |  |  | ● | ● | ● | ◆ |
| Optional demo / workshop paper preparation | △ | △ | △ | △ | △ |  |  |  |  |  |  |  |  |

### 3.1 Timeline Rationale

Weeks 1–2 focus on literature review, research framing, ethics preparation, and system design. Since the project involves human participants, ethics preparation must begin early and formal participant data collection must not begin before approval.

Weeks 2–6 focus on implementation. The first technical goal is a local simulation, followed by multimodal API testing and robot integration. The technical framework is scheduled to be completed by the end of June or early July, leaving sufficient time for pilot testing, formal participant evaluation, data analysis, and dissertation writing.

Weeks 5–7 are reserved for pilot testing and protocol refinement. Pilot testing is necessary because the system must be checked before formal data collection, especially for speech recognition reliability, visual input quality, prompt appropriateness, robot timing, and safe physical setup.

Weeks 7–9 are reserved for formal participant evaluation. At least two weeks are allocated because participant recruitment, scheduling, running sessions, collecting questionnaires, and organising data are likely to require more than a single week.

Weeks 9–11 focus on result analysis, while Weeks 10–13 focus on dissertation consolidation and final editing. Dissertation writing and experiment logging run throughout the whole project to reduce end-stage writing risk.

---

## 4. Operational Aspects

### 4.1 Data Management Plan

The project will collect several types of research data:

- system logs;
- robot intervention timestamps;
- questionnaire responses;
- optional short interview notes;
- audio recordings or transcripts of participant conversations, if approved;
- visual context summaries produced by the system, if approved.

All raw data and processed data will be stored separately. Each session will be assigned an anonymised participant-pair ID. Personal identifiers, consent forms, and contact details will be stored separately from study data.

The planned data structure will be:

```text
project/
    data/
        raw_audio_or_transcripts/
        anonymised_transcripts/
        questionnaire_responses/
        system_logs/
        visual_context_summaries/
    src/
        robot_control/
        dialogue_manager/
        multimodal_context/
        evaluation/
    configs/
    results/
    figures/
    dissertation/
    ethics_documents/
```

The repository will include source code, configuration files, documentation, and non-sensitive examples. Raw participant data, audio recordings, consent forms, identifiable information, and any images or video frames containing participants will not be committed to version control.

A data log will record:

- date and location of the session;
- participant pair ID;
- experimental condition;
- system version;
- whether speech, transcript, image, or video data was used;
- robot intervention timestamps;
- questionnaire completion status;
- anonymisation status;
- technical issues or deviations from protocol.

If third-party AI, transcription, or multimodal API tools are used, participant-facing documents will explain what data is processed, by which type of service, and whether the data can be withdrawn after processing. Where possible, the system will avoid sending identifiable images, video, or raw conversations to external services. If external processing is necessary, it will only be used when permitted by the ethics approval and clearly disclosed to participants.

### 4.2 Ethical Statement

This project involves human participants and therefore will require ethics approval before formal data collection. The study will not begin until the appropriate approval has been obtained. Retrospective approval is not possible.

Participants will receive an information sheet explaining the project purpose, procedure, data collected, risks, storage, withdrawal rights, and contact details. Written consent will be obtained before participation. Participants will be told that the robot is a research prototype and that their interaction may be logged or recorded only if they consent.

The main ethical risks are:

- mild social awkwardness when speaking with a stranger;
- possible discomfort from being observed or recorded;
- privacy risks from conversation transcripts, audio data, or visual data;
- risk of participants feeling judged by the robot or the system;
- risk of overclaiming the system’s social benefit;
- risk that third-party AI tools may process data in ways participants do not expect.

Mitigation measures include:

- allowing participants to stop at any time;
- avoiding sensitive or personal conversation prompts;
- using safe and neutral ice-breaking topics;
- minimising raw audio, image, and video storage where possible;
- anonymising logs and questionnaire data;
- avoiding unauthorised mass email recruitment;
- not using unapproved third-party tools for identifiable participant data;
- reporting results cautiously and acknowledging sample-size limitations.

The visual component will be limited to broad interaction-relevant cues. The project will not attempt to infer sensitive characteristics such as ethnicity, mental health, personality, social anxiety, or cultural background from visual input.

### 4.3 Health and Safety Statement

The project is low physical risk. The main risks are related to normal computer use, use of robotic hardware, and running a short controlled interaction study in a lab or university room.

These risks will be managed through:

- appropriate room setup;
- safe cable management;
- supervised use of the robot;
- ensuring that the robot does not collide with participants;
- keeping a safe distance between the robot and participants;
- regular breaks during development work;
- avoiding physically demanding participant tasks.

Participants will not be asked to perform physically demanding activities. The robot will be positioned safely and will not be allowed to move in ways that could cause physical risk. If a tablet or laptop is used, it will be placed securely.

### 4.4 Environmental Impact Statement

The environmental impact of this project is expected to be limited. The main environmental costs are electricity use for laptops, robot operation, and possible cloud or API usage for large-model inference.

The project will minimise unnecessary computation by:

- using existing model APIs rather than training large models;
- testing the system first with small local prototypes;
- limiting repeated API calls during development;
- caching non-sensitive prompts and logs where appropriate;
- avoiding large-scale model training;
- stopping optional enhancements if they do not directly support the evaluation.

The project’s research question is about the design and evaluation of a working robotic facilitator, not about training a new large model. This keeps the computational requirements realistic for an MSc project.

---

## 5. Project Impact, Commercial and Legal Aspects

### 5.1 Sustainability, Ethical and Social Impact

The project addresses a socially relevant problem: people may experience awkwardness, hesitation, or social anxiety when starting conversations with strangers. A robotic facilitator may provide a socially safe “third object” or light prompt that helps people begin or continue conversation. This could be useful in settings such as university welcome events, public waiting areas, student communities, conferences, museums, or elderly care environments.

However, the project must avoid overclaiming. A robot cannot be assumed to improve all conversations. Some users may find robotic intervention intrusive, unnatural, childish, or distracting. The evaluation will therefore consider both positive and negative reactions.

The design will prioritise non-sensitive prompts, participant comfort, and the ability to disengage. It will also consider inclusion issues, because users from different cultural backgrounds may have different expectations about interruption, humour, eye contact, personal space, and robotic behaviour.

The project will treat the robot as a facilitation tool, not as a replacement for human social skill, counselling, or professional support.

### 5.2 Commercial Aspects

There may be commercial potential for robotic or AI-supported facilitation systems in universities, conferences, museums, public spaces, hospitality, and care settings. A deployable version could help create low-pressure social interaction opportunities, support new student integration, or encourage participation in small-group activities.

However, the commercial value of such a system depends less on technical novelty alone and more on reliability, safety, social acceptability, privacy protection, and evidence that it improves interaction without becoming intrusive. A system that frequently interrupts, misreads visual context, or generates awkward prompts would not be suitable for deployment.

A realistic commercial endpoint would not simply be a “chatbot robot”, but a configurable facilitation platform that allows organisations to choose conversation settings, safe prompt libraries, interaction duration, logging options, privacy settings, and deployment contexts. The project’s contribution to this broader direction is an early prototype and evaluation evidence about whether such a robotic facilitator can be useful in a small controlled setting.

### 5.3 Legal Aspects

The project may involve personal data if participant audio, video, images, transcripts, questionnaire responses, or consent forms are collected. Therefore, data protection principles must be considered, including transparency, minimisation, secure storage, retention, and participant rights.

The project will ensure that:

- participants understand what data is collected;
- participants give informed consent;
- identifiable data is stored securely;
- data is anonymised or pseudonymised where possible;
- retention periods are clearly defined;
- recordings are not shared publicly;
- third-party tools are not used for personal data unless approved;
- participants are informed if any AI or cloud service processes their data.

If the system uses commercial LLM APIs, multimodal APIs, speech-to-text tools, or cloud services, their terms of use, data processing arrangements, and privacy implications will be checked before participant data is processed. Identifiable participant conversation data or images will not be sent to external services unless this is necessary, approved, and explicitly disclosed.

The project also has intellectual property considerations. The code, prompts, robot control scripts, questionnaire materials, and analysis outputs should be clearly documented. If a short demo paper or workshop submission is prepared, authorship and contribution should be agreed before submission.

---

## 6. Risk Management and Fallback Plan

| Risk | Impact | Mitigation |
|---|---|---|
| Ethics approval takes longer than expected | Delays participant study | Start ethics preparation in Week 1; prepare participant information, consent form, questionnaire, risk assessment, and data management plan early. |
| Robot integration is unreliable | Formal evaluation may fail | Build a local simulation first; use semi-structured operation or Wizard-of-Oz support if full autonomy is unstable. |
| Speech recognition performs poorly | Conversation state tracking may be inaccurate | Use typed input, manual transcription, push-to-talk, or simplified logging during evaluation if needed. |
| Visual input or multimodal API is unreliable | Multimodal component may not support decisions well | Keep the core system functional with speech and conversation history; use visual context only as an auxiliary cue. |
| Third-party API cannot process participant data under ethics constraints | Multimodal function may be restricted | Use non-identifiable test examples during development; disable visual API calls during formal evaluation if necessary. |
| Recruitment is difficult | Small sample size | Use approved recruitment routes, posters, opt-in sign-up, and supervisor-approved channels; focus on controlled qualitative and descriptive analysis if sample size is limited. |
| Robot interventions feel awkward or intrusive | Poor user ratings | Treat this as a valid finding; analyse when and why the robot was intrusive or ineffective. |
| Optional tablet interface takes too long | System complexity increases | Keep the tablet display optional; the core system depends on robot speech and simple behaviours. |
| Advanced adaptive intervention logic is not completed | Reduced technical sophistication | Keep advanced proactive scoring as future work; prioritise working prototype and valid user evaluation. |
| Time is limited | Not all enhancements possible | Prioritise the minimum viable version: speech/transcript input, limited visual context if feasible, simple facilitation rules, robot output, logging, and with/without robot evaluation. |

---

## 7. Minimal Viable Version and Full Version

### 7.1 Minimal Viable Version

The minimal viable version will include:

- a local conversation simulation;
- speech or transcript input from two participants;
- conversation history tracking;
- simple interaction state rules based on silence, turn-taking, and short replies;
- safe prompt generation;
- robot speech output;
- basic logging;
- with/without robot user evaluation;
- questionnaire-based subjective feedback.

This version is sufficient to answer the core research question because it provides a working robotic facilitator and allows comparison with a no-robot condition.

### 7.2 Full Version

The full version will additionally include:

- limited visual context interpreted through a multimodal API;
- improved robot gestures or facial expressions;
- optional tablet display for topic cards or visual prompts;
- richer intervention categories;
- optional scripted-prompt baseline;
- short demo video or workshop/demo paper preparation.

The full version will only be pursued after the core system and evaluation protocol are stable.

---

## 8. Expected Outcomes

The expected outcomes of this project are:

1. a working multimodal robotic facilitation prototype;

2. a documented technical framework for combining speech, conversation history, limited visual context, response generation, and embodied robot output;

3. a small-scale user evaluation comparing conversation with and without the robotic facilitator;

4. objective evidence about conversation duration, silence, speaking turns, and robot interventions;

5. subjective evidence about participant comfort, awkwardness, naturalness, helpfulness, and intrusiveness;

6. a discussion of limitations, ethical issues, and future improvements for robot-mediated human-human conversation.

The final dissertation will not claim that the system generalises to all public spaces or all user groups. Instead, it will present a realistic prototype and an evidence-based analysis of its effect in a controlled interaction setting.
