import os
import threading

from kivy.app import App
from kivy.clock import Clock
from kivy.lang import Builder
from kivy.properties import StringProperty
from kivy.uix.boxlayout import BoxLayout

from assistant import MrRAssistant


KV_FILE = "mr_r.kv"


class MrRInterface(BoxLayout):
    status = StringProperty("Mr.R is ready")
    response_text = StringProperty(
        "Hello. I am Mr.R, your personal AI assistant."
    )

    def __init__(self, **kwargs):
        super().__init__(**kwargs)

        self.assistant = MrRAssistant(
            api_key=os.environ.get("OPENAI_API_KEY", "")
        )

    def add_message(self, text):
        self.ids.chat.text += text + "\n\n"

    def ask(self, text=None):
        if text is None:
            text = self.ids.user_input.text.strip()

        if not text:
            return

        self.ids.user_input.text = ""
        self.add_message(f"You: {text}")
        self.status = "Mr.R is thinking..."

        threading.Thread(
            target=self._process,
            args=(text,),
            daemon=True
        ).start()

    def _process(self, text):
        try:
            result = self.assistant.process(text)

            Clock.schedule_once(
                lambda dt: self._show_result(result)
            )

        except Exception as e:
            Clock.schedule_once(
                lambda dt: self._show_error(str(e))
            )

    def _show_result(self, result):
        self.add_message(f"Mr.R: {result}")
        self.status = "Mr.R is ready"

    def _show_error(self, error):
        self.add_message(f"Mr.R: Sorry, something went wrong.\n{error}")
        self.status = "Error"

    def clear_chat(self):
        self.ids.chat.text = ""


class MrRApp(App):
    title = "Mr.R - Virtual AI Assistant"

    def build(self):
        return Builder.load_file(KV_FILE)


if __name__ == "__main__":
    MrRApp().run()
    # ai-voice-assistant
A comprehensive AI voice assistant with speech recognition, NLP, text-to-speech, and command execution capabilities
