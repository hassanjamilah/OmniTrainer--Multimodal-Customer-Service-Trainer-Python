### Project Overview
Overview
This is an AI-powered content moderation system for customer service interactions at a fictional company called ACME Enterprise.

What it does:

Moderates text, images, videos, and audio before they're sent to customers
Detects issues like: PII (personally identifiable information), unprofessional tone, unfriendly content, disturbing images/videos, and low-quality media
Blocks harmful content and provides detailed explanations for why content was flagged
The app allows a trainee customer agent to interact with a simulated angry customer, played by an LLM. The LLM-customer has bought a product from ACME (the ACME Power Widget Pro) and the product stopped working. The customer agent needs to handle this case by chatting with the customer, and every message and content provided by the trainee agent is moderated and observed to make sure all communications following company standards.

Architecture:

Specialized Agents - Four moderation agents (text, image, video, audio), each using Google Gemini AI with custom prompts to check their specific content type (agents are in the agents folder)
LLM-as-a-customer: an agent (agents/customer_agent.py)
Structured Results - Each agent returns a Pydantic model with specific flags (e.g., contains_pii, is_unfriendly) plus a rationale. The definitions are in the the types/moderation_result.py folder
Frontend: Gradio Chat UI - Interactive web interface where users can chat and upload files, with real-time moderation. This is gradio_app.py.
Backend: FastAPI REST API - HTTP endpoints for programmatic access (/moderate/text, /moderate/image, etc.). These are provided as-is. They just wrap in HTTP endpoints the functionality offered by the agents. This is fastapi_app.py. The division in frontend/backend services is typical of web applications, and allow different front-ends to utilize the same services from the backend. For example, in the hypothetical scenario of this app, after the initial PoC phase using the Gradio app, we might want to move to a more production-grade React/Vue/Angular app. This new app can use the same backend, and the two apps can even co-exist for a time until the new app is proved to work. No change is needed in the AI agents or on the backend.
Observability - Phoenix integration for tracing and monitoring AI agent behavior. Some setup is in tracing.py.
A convenience executable that starts the 3 services: the backend (fastAPI APIs), the frontend (the gradio app) as well as Arize Phoenix for tracing.
How to work on the project
The scaffolding is provided to you. You will complete key parts of the code, applying what you have learned, following a precise sequence of steps described below.

Every step needs to be executed in order. You can self-verify that the step has been completed successfully by executing the tests that we will indicate (already provided to you).

DO NOT skip steps and do not proceed if the tests are not passing, as the next steps build on the previous one.

### Instructions
Execution
Follow the following steps in order:

1. Moderation result
Here we setup the Pydantic models for our moderation agents. These are the output schemas for the text, audio, image and video moderation agents we'll work on later.

Edit the types/moderation_result.py file and replace the # TODO sections with your code
Run the test to get feedback on your work. From the root of the starter run: uv run pytest tests/test_moderation_result.py -vv. If you get an error, read it carefully as it will give you information on what to fix.
DO NOT continue unless the test passes.

2. Text agent
Here we will complete the setup of the moderation agent for text.

Edit the text agent (agents/text_agent.py) and replace the # TODO sections
Run the test to get feedback on your work. From the root of the starter run: uv run pytest tests/test_text_agent.py -vv. If you get an error, read it carefully as it will give you information on what to fix.
DO NOT continue unless the test passes.

3. Image agent
Here we will complete the setup of the moderation agent for images, similarly to what we just did for text. Since this is a multimodal agent, we will use the multimodal capabilities of Pydantic AI and Gemini.

Edit the image agent (agents/image_agent.py) and replace the # TODO sections
Run the test to get feedback on your work. From the root of the starter run: uv run pytest tests/test_image_agent.py -vv. If you get an error, read it carefully as it will give you information on what to fix.
DO NOT continue unless the test passes.

4. Video and Audio agents
The audio and video agents are already completed. You will just test them to make sure they work.

The Video and Audio agents are already complete. You can test them with uv run pytest tests/test_video_agent.py and uv run pytest tests/test_audio_agent.py.

5. Gradio App
Here we put everything together into a Gradio app, which constitutes the front-end of our solution.

Edit the gradio app (gradio_app.py) and replace all # TODO sections. There are quite a few, going from the top to the bottom. Before moving on, make sure you did not left any by searching TODO in the file (Crtl+F or Command+F).
Run the test to get feedback on your work. From the root of the starter run: uv run pytest tests/test_gradio_app.py -vv. If you get an error, read it carefully as it will give you information on what to fix.
DO NOT continue unless the test passes.

6. Run all tests
It is now time to verify your work. Run the project's tests at once with:

uv run pytest tests/ -vv --ignore=tests/test_env_setup.py
All of them should pass. If any fail, fix them before submitting your project — and keep this output, it is what goes in your submission's evidence file.

What these tests check. None of them call Gemini (the agent tests use a stand-in model), so they cost nothing and check your wiring: that each agent is set up and connected correctly. They still need a .env with non-empty GEMINI_API_KEY, USER_API_KEY and DEFAULT_GOOGLE_MODEL, or they stop with ValueError: … required but not set.

tests/test_env_setup.py is excluded because it calls Gemini for real to check your own setup, as do the evals in step 7, which measure how well your agents judge content. Once your credentials are in place you can include it by dropping the --ignore:

uv run pytest tests/ -vv
7. Evals
Now that our system is complete, we need to run evaluations to understand and measure how it behaves.

Edit the evals/text/test_cases.py and replace all # TODO sections. Same for evals/image/test_cases.py.
Once you are done, run uv run evals/text/test_cases.py to see the results of the evals on text. NOTE: your score will NOT always be 100%! that is not a bug. The text agent is not perfect!
Run the evals for the other media (uv run evals/image/test_cases.py, uv run evals/audio/test_cases.py and uv run evals/video/test_cases.py)
Reading your eval results
Unlike the tests, evals call Gemini for real — for each case, once for the agent being evaluated and once more for an LLM judge (LLMJudge) that grades the rationale the agent wrote. Both are language models, so the same case can pass on one run and fail on the next. Measuring that variation is what evals are for, and it is why a score below 100% is expected and is not a defect in your submission.

EVAL_NUM_REPEATS in your .env sets how many times each case is run. env.example sets it to 1, which is quick and cheap — keep it there while you are still writing your test cases. Raising it (to 5, say) gives a steadier score but makes that many times as many API calls, and in the workspace those come out of your voc- key's budget. (If your .env says 5, it was copied from an older env.example.) Each eval run reads .env fresh, so there is nothing to restart.

What to look for: every case runs without an error, your unacceptable cases are mostly flagged, and your acceptable ones mostly are not. When a case fails, read the agent's rationale in the output — it usually tells you whether the agent or your expected value is wrong.

Play with the app
Conversation
Now that you are done, the app should work.

In the workspace, first run this in the terminal so that the Phoenix dashboard can display through the workspace proxy (see Opening the app and Phoenix in your browser on the Environment Setup page). Skip it on your own machine.

export PHOENIX_HOST_ROOT_PATH=/proxy/6006
Then start the app:

uv run multimodal-moderation
This starts three services: the chat app on port 7860, the backend API on port 8000, and Phoenix on port 6006. Wait until the terminal shows a Phoenix UI: line and a Running on local URL line, then open the chat app — on your own machine at http://localhost:7860/; in the workspace at your VSCODE_PROXY_URI address with 7860 in place of {{port}}. You are free to play with it, however, if you need inspiration, this is how a conversation could go:

YOU> Welcome to ACME Customer Service. How can I help?
CUSTOMER-LLM> ... [will complain about the product not working]
YOU> I am sorry to hear that. Is this the product you are talking about? [attach evals/test_data/professional_image.jpg]
CUSTOMER-LLM> ... [will say something about wanting a refund]
YOU> I am sorry but I absolutely cannot offer a refund
------> message will be flagged as rude
YOU> I am going to help you solving your issue. I am authorized to offer you a replacement. Would you be willing to accept it?
CUSTOMER-LLM> ... [probably won't accept]
[you can now close the conversation by clicking on the End Conversation button]
See your traces
After a conversation like the one suggested above, open the Phoenix UI — on your own machine at http://localhost:6006/projects; in the workspace at your VSCODE_PROXY_URI address with 6006 in place of {{port}}, followed by projects. (In the workspace the terminal prints Phoenix UI: http://localhost:6006/proxy/6006 — that address is the workspace's own and will not open in your browser.) Click on the default project and see your traces and spans. Explore the different metadata and different reports. This is the view to screenshot for your evidence file.

If the Phoenix page is blank in the workspace, the app was started without PHOENIX_HOST_ROOT_PATH. Stop it with Ctrl+C, run export PHOENIX_HOST_ROOT_PATH=/proxy/6006, start it again, and have a new conversation so there are traces to look at.

See your backend APIs
Go to http://localhost:8000/docs to see a nice documentation of your moderation APIs. (In the workspace, use your VSCODE_PROXY_URI address with 8000 in place of {{port}}, followed by docs. If it shows "Failed to load API definition", your workspace was set up before this was fixed: skip this part, as it is not needed for your submission.) If you wanna test them out from here, click on Authorize on the upper right and insert your USER_API_KEY (the one you have set in your .env file). Then click on an endpoint (say, /api/v1/moderate-text) and click on Try it out (in the upper right). You will see a JSON like:

{
  "text": "string"
}
Just change string to a message and click Execute, your message will be moderated. Scroll down a bit to see the results.

Stopping and restarting the app
uv run multimodal-moderation keeps running until you stop it. It uses four ports: 7860 (chat app), 8000 (backend API), 6006 (Phoenix dashboard and trace collector) and 4317 (Phoenix's second trace collector).

To stop it, click into the terminal where it is running and press Ctrl+C. That stops all three services. Closing the browser tab does not stop them, and closing the terminal can leave them running.
Restart after every .env change: Ctrl+C, then uv run multimodal-moderation again. The app reads .env only when it starts. Restart after changing agent or chat-app code, too, before you try it in the app (the tests always pick up your latest code on their own).
"Address already in use" — for example Failed to bind to address [::]:4317, or the same for port 6006, 7860 or 8000 — means an earlier copy is still running: usually one started in a terminal you closed, or a Phoenix demo notebook from an earlier lesson that is still open. Stop the old copy of the app with:
pkill -f multimodal-moderation
If the error persists, a notebook is probably holding the port: shut down its kernel (in Jupyter:

Kernel → Shut Down Kernel*). Then start the app again.
In a new terminal (in the workspace), first run cd /voc/work/code/project/starter/ and source .venv/bin/activate — otherwise uv is not found — then`` export PHOENIX_HOST_ROOT_PATH=/proxy/6006, then start the app.
Submission Instructions
You can use the workspace to submit, package your files as a ZIP, or use a GitHub repository.

Alongside your code, include one evidence file at the top level named RUN_EVIDENCE.pdf containing all four of the following:

Pasted as text — the output of uv run pytest tests/ -vv --ignore=tests/test_env_setup.py. This one command covers every test the rubric refers to.
Pasted as text — the output of one eval run, for example uv run evals/text/test_cases.py. Remember that a score below 100% is expected and is not a defect.
A screenshot — one conversation in the Gradio UI where a message was flagged by moderation, showing the rationale as it was displayed to you.
A screenshot — your traces in the Phoenix dashboard, showing the span names. In the workspace, open Phoenix as described in See your traces above; if the page is blank, follow the note there before you take the screenshot.
Why the formats differ, and where there is room. Items 1 and 2 are things a tool printed, so pasted text is better than a picture of it — your reviewer can read and search it, and you can copy it in one step. If you have already screenshotted that output and it is legible, that is fine too. Items 3 and 4 are different: they are things only a screen can show — that the interface really displayed the flag and its rationale, and that your spans really reached Phoenix. For those two, a description in words shows only that you know what should have happened, so a screenshot is required.

A single PDF is the safest container. You may submit RUN_EVIDENCE.md with the screenshots attached alongside it instead, but loose .png or .jpg files may not appear in your reviewer's file list.

tests/test_env_setup.py is excluded from the command above because it checks your own credentials and calls the Gemini API for real.

This file is part of the submission, not an extra: your reviewer checks your code and your evidence, and a submission without it will come back for another attempt.

### Rubric
Use this project rubric to understand and assess the project criteria.

Structured Moderation Outputs
Criteria	Submission Requirements
The project defines and uses a structured moderation output model

The base ModerationResult model defines only the shared rationale: str field (required, no default).
Each modality subclass defines its own flags on the correct class and does not push modality-specific flags onto the base: text -> contains_pii, is_unfriendly, is_unprofessional ; image/video -> contains_pii, is_disturbing, is_low_quality ; audio per the student's AudioModerationResult TODO.
Flag fields are typed bool and required, with no default, exactly as the provided TextModerationResult, ImageModerationResult and VideoModerationResult classes define them ; rationale is required.
The correct model is imported and used by each corresponding moderation agent.
All unit tests in tests/test_moderation_result.py pass, unmodified.
Moderation Agents
Criteria	Submission Requirements
The project includes a functioning text moderation agent that analyzes user messages

The text moderation agent in agents/text_agent.py uses Gemini or a similar LLM to detect PII, unprofessional tone, and unfriendly content.
The agent returns results as a valid ModerationResult instance.
The model parsing and response logic work as intended.
All tests in tests/test_text_agent.py pass, unmodified — the test output in the evidence file covers this.
The project includes an image moderation agent that processes binary inputs and returns structured results

The image moderation agent in agents/image_agent.py correctly wraps uploaded image bytes into a BinaryContent object.
The agent uses Gemini or an equivalent LLM to identify disturbing or inappropriate visual content.
The output conforms to the ModerationResult model.
All tests in tests/test_image_agent.py pass, unmodified — the test output in the evidence file covers this.
Multimodal Moderation Interface
Criteria	Submission Requirements
The project includes a Gradio Chat UI that handles multimodal input and integrates moderation

The gradio_app.py implements a gr.ChatInterface supporting both text and file uploads.
All user messages and uploaded files are moderated before being displayed.
The chat interface maintains conversation state between user and customer messages.
All TODOs related to moderation handling are completed.
tests/test_gradio_app.py passes, unmodified — the test output in the evidence file covers this.
The project integrates a simulated LLM customer agent that participates in conversations

The customer_agent.run(...) call is implemented asynchronously and used to generate dynamic customer responses.
The prior conversation is passed to the customer agent (the past_messages_state TODO in gradio_app.py), so replies stay contextually consistent.
The moderation path is wired so that a flagged message is shown with its rationale and is not sent to the customer agent.
The evidence file includes a screenshot of one conversation in the Gradio UI that included a flagged moderation event, showing the rationale as it was displayed.
Observability and Tracing
Criteria	Submission Requirements
The project includes complete tracing instrumentation with OpenTelemetry and Arize Phoenix

Tracing spans are implemented for "moderate_text", "chat_turn", "conversation" (with "session.id"), and "feedback".
The spans are created using the tracer defined in tracing.py.
Each span includes relevant attributes, such as session identifiers or feedback content.
The evidence file includes a screenshot of the Phoenix dashboard showing these spans from a run.
Moderation Evals
Criteria	Submission Requirements
The project defines and executes evaluation cases for moderation agents using Pydantic Evals

The evals/text/test_cases.py and evals/image/test_cases.py files include test cases covering both acceptable and unacceptable content.
Evals use structured inputs and return results consistent with ModerationResult.
Every TODO in both files is filled in, including the expected-flag values for each case.
The evidence file includes the output of one eval run. A score below 100% is expected and is not a defect.
Suggestions to Make Your Project Stand Out
Extend moderation capabilities to include additional flags (e.g., hate speech, spam, or misinformation).

Add visual analytics or dashboards summarizing flagged content using backend APIs.

Introduce persona-based variations in the LLM customer agent to simulate different emotional tones or scenarios.


