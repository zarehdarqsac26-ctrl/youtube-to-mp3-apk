import os
import threading

os.environ.setdefault("KIVY_NO_CONSOLELOG", "1")

import yt_dlp
from kivy.app import App
from kivy.clock import Clock
from kivy.core.window import Window
from kivy.graphics import Color, RoundedRectangle
from kivy.logger import Logger
from kivy.metrics import dp
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.button import Button
from kivy.uix.floatlayout import FloatLayout
from kivy.uix.label import Label
from kivy.uix.popup import Popup
from kivy.uix.textinput import TextInput
from kivy.uix.widget import Widget


BACKGROUND_COLOR = (0.85, 0.86, 0.89, 1)
DARK_BACKGROUND = (0.07, 0.07, 0.09, 1)
LIGHT_SURFACE = (0.95, 0.95, 0.97, 1)
DARK_SURFACE = (0.14, 0.14, 0.17, 1)
LIGHT_TEXT = (0.08, 0.08, 0.10, 1)
DARK_TEXT = (0.96, 0.96, 0.98, 1)
STATUS_LIGHT_TEXT = (0.02, 0.02, 0.025, 1)
ARMENIAN_FONT = r"C:\Windows\Fonts\segoeui.ttf"
SYMBOL_FONT = r"C:\Windows\Fonts\seguisym.ttf"
LANGUAGE_DATA = {
    "English": {
        "code": "EN",
        "flag": ((0.10, 0.25, 0.55, 1), (1, 1, 1, 1), (0.75, 0.12, 0.18, 1)),
        "title": "YouTube to MP3",
        "subtitle": "Paste the video link",
        "url_hint": "YouTube URL",
        "ready": "Ready",
        "download": "DOWNLOAD MP3",
        "footer": "The file will be saved in the app Downloads folder",
        "allowed": "Use permitted links only",
        "empty": "Paste a YouTube URL",
        "downloading": "Downloading...",
        "success": "Ready. MP3 saved.",
        "error": "Error",
    },
    "Հայերեն": {
        "code": "HY",
        "flag": ((0.15, 0.35, 0.75, 1), (0.95, 0.65, 0.18, 1), (0.75, 0.12, 0.16, 1)),
        "title": "YouTube դեպի MP3",
        "subtitle": "Տեղադրիր տեսանյութի հղումը",
        "url_hint": "YouTube URL",
        "ready": "Պատրաստ է",
        "download": "ՆԵՐԲԵՌՆԵԼ MP3",
        "footer": "Ֆայլը կպահվի հավելվածի Downloads պանակում",
        "allowed": "Օգտագործիր միայն թույլատրված հղումներ",
        "empty": "Տեղադրիր YouTube URL",
        "downloading": "Ներբեռնում է...",
        "success": "Պատրաստ է։ MP3-ը պահպանվեց։",
        "error": "Սխալ",
    },
    "Español": {
        "code": "ES",
        "flag": ((0.75, 0.12, 0.16, 1), (0.95, 0.75, 0.20, 1), (0.75, 0.12, 0.16, 1)),
        "title": "YouTube a MP3",
        "subtitle": "Pega el enlace del video",
        "url_hint": "URL de YouTube",
        "ready": "Listo",
        "download": "DESCARGAR MP3",
        "footer": "El archivo se guardará en la carpeta Downloads",
        "allowed": "Usa solo enlaces permitidos",
        "empty": "Pega una URL de YouTube",
        "downloading": "Descargando...",
        "success": "Listo. MP3 guardado.",
        "error": "Error",
    },
    "Русский": {
        "code": "RU",
        "flag": ((1, 1, 1, 1), (0.15, 0.35, 0.75, 1), (0.75, 0.12, 0.16, 1)),
        "title": "YouTube в MP3",
        "subtitle": "Вставьте ссылку на видео",
        "url_hint": "Ссылка YouTube",
        "ready": "Готово",
        "download": "СКАЧАТЬ MP3",
        "footer": "Файл сохранится в папке Downloads приложения",
        "allowed": "Используйте только разрешенные ссылки",
        "empty": "Вставьте ссылку YouTube",
        "downloading": "Скачивание...",
        "success": "Готово. MP3 сохранен.",
        "error": "Ошибка",
    },
    "Français": {
        "code": "FR",
        "flag": ((0.15, 0.30, 0.70, 1), (1, 1, 1, 1), (0.75, 0.12, 0.16, 1)),
        "title": "YouTube vers MP3",
        "subtitle": "Collez le lien de la vidéo",
        "url_hint": "URL YouTube",
        "ready": "Prêt",
        "download": "TÉLÉCHARGER MP3",
        "footer": "Le fichier sera enregistré dans le dossier Downloads",
        "allowed": "Utilisez uniquement des liens autorisés",
        "empty": "Collez une URL YouTube",
        "downloading": "Téléchargement...",
        "success": "Prêt. MP3 enregistré.",
        "error": "Erreur",
    },
}
Window.clearcolor = BACKGROUND_COLOR


class NeumorphicCard(BoxLayout):
    def __init__(self, **kwargs):
        self.surface_color = LIGHT_SURFACE
        self.dark_mode = False
        super().__init__(**kwargs)
        self.bind(pos=self.update_canvas, size=self.update_canvas)

    def update_canvas(self, *_):
        self.canvas.before.clear()
        with self.canvas.before:
            light = (0.25, 0.25, 0.29, 0.8) if self.dark_mode else (1, 1, 1, 0.9)
            dark = (0.02, 0.02, 0.03, 0.9) if self.dark_mode else (0.78, 0.80, 0.83, 0.8)
            Color(*light)
            RoundedRectangle(pos=(self.x - 6, self.y + 6), size=self.size, radius=[40])
            Color(*dark)
            RoundedRectangle(pos=(self.x + 6, self.y - 6), size=self.size, radius=[40])
            Color(*self.surface_color)
            RoundedRectangle(pos=self.pos, size=self.size, radius=[40])


class NeumorphicInputField(BoxLayout):
    def __init__(self, **kwargs):
        self.surface_color = (0.89, 0.90, 0.92, 1)
        self.dark_mode = False
        super().__init__(**kwargs)
        self.bind(pos=self.update_canvas, size=self.update_canvas)

    def update_canvas(self, *_):
        self.canvas.before.clear()
        with self.canvas.before:
            shadow = (0.02, 0.02, 0.03, 0.7) if self.dark_mode else (0.78, 0.80, 0.83, 0.6)
            edge = (0.23, 0.23, 0.28, 0.8) if self.dark_mode else (1, 1, 1, 0.7)
            Color(*shadow)
            RoundedRectangle(pos=self.pos, size=self.size, radius=[15])
            Color(*edge)
            RoundedRectangle(pos=(self.x + 2, self.y - 2), size=(self.width - 2, self.height - 2), radius=[15])
            Color(*self.surface_color)
            RoundedRectangle(pos=(self.x + 2, self.y + 2), size=(self.width - 4, self.height - 4), radius=[15])


class NeumorphicButton(Button):
    def __init__(self, **kwargs):
        self.surface_color = LIGHT_SURFACE
        self.dark_mode = False
        self._pressed = False
        super().__init__(**kwargs)
        self.background_color = (0, 0, 0, 0)
        self.bind(pos=self.update_canvas, size=self.update_canvas)

    def update_canvas(self, *_):
        self.canvas.before.clear()
        with self.canvas.before:
            offset = 1 if self._pressed else 4
            light = (0.25, 0.25, 0.29, 0.8) if self.dark_mode else (1, 1, 1, 0.9)
            dark = (0.02, 0.02, 0.03, 0.9) if self.dark_mode else (0.78, 0.80, 0.83, 0.8)
            Color(*light)
            RoundedRectangle(pos=(self.x - offset, self.y + offset), size=self.size, radius=[12])
            Color(*dark)
            RoundedRectangle(pos=(self.x + offset, self.y - offset), size=self.size, radius=[12])
            Color(*self.surface_color)
            RoundedRectangle(pos=self.pos, size=self.size, radius=[12])

    def on_touch_down(self, touch):
        if self.collide_point(*touch.pos):
            self._pressed = True
            self.update_canvas()
        return super().on_touch_down(touch)

    def on_touch_up(self, touch):
        self._pressed = False
        self.update_canvas()
        return super().on_touch_up(touch)


class SettingsPanel(BoxLayout):
    # Kivy-ի popup style-ի համար անհրաժեշտ դատարկ title հատկությունը։
    title = ""

    def __init__(self, dark_mode, **kwargs):
        self.dark_mode = dark_mode
        self.surface_color = DARK_SURFACE if dark_mode else LIGHT_SURFACE
        super().__init__(**kwargs)
        self.padding = dp(22)
        self.spacing = dp(14)
        self.orientation = "vertical"
        with self.canvas.before:
            self.bg_color = Color(*self.surface_color)
            self.background = RoundedRectangle(radius=[dp(22)])
        self.bind(pos=self._update_background, size=self._update_background)

    def _update_background(self, *_):
        self.background.pos = self.pos
        self.background.size = self.size


class FlagBadge(Widget):
    def __init__(self, colors, **kwargs):
        super().__init__(**kwargs)
        self.flag_colors = colors
        self.bind(pos=self._update_flag, size=self._update_flag)

    def _update_flag(self, *_):
        self.canvas.clear()
        band_height = self.height / 3
        with self.canvas:
            for index, color in enumerate(self.flag_colors):
                Color(*color)
                RoundedRectangle(
                    pos=(self.x, self.y + band_height * (2 - index)),
                    size=(self.width, band_height + 1),
                    radius=[dp(3)],
                )


class DownloaderScreenUI(FloatLayout):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.dark_mode = False
        self.text_widgets = []
        self.current_language = "English"

        # Վերևի ձախ compact լեզվի կոճակը։
        self.language_button = NeumorphicButton(
            text="EN  English",
            font_name=ARMENIAN_FONT,
            font_size="13sp",
            color=LIGHT_TEXT,
            size_hint=(None, None),
            size=(dp(155), dp(42)),
            pos_hint={"x": 0.03, "top": 0.96},
        )
        self.language_button.bind(on_press=self.open_language_menu)
        self.add_widget(self.language_button)

        self.card = NeumorphicCard(
            orientation="vertical",
            size_hint=(0.85, None),
            height=390,
            pos_hint={"center_x": 0.5, "center_y": 0.5},
            padding=[25, 30, 25, 30],
            spacing=15,
        )
        self.add_widget(self.card)

        self.settings_button = NeumorphicButton(
            text="⚙",
            font_name=SYMBOL_FONT,
            font_size="22sp",
            color=LIGHT_TEXT,
            size_hint=(None, None),
            size=(48, 48),
            pos_hint={"right": 0.97, "top": 0.96},
        )
        self.settings_button.bind(on_press=self.open_settings)
        self.add_widget(self.settings_button)

        self.title_label = Label(
            text="YouTube to MP3",
            font_name=ARMENIAN_FONT,
            font_size="31sp",
            bold=True,
            color=LIGHT_TEXT,
            size_hint_y=None,
            height=40,
        )
        self.add_text(self.title_label)

        self.subtitle_label = Label(
            text="Paste the video link",
            font_name=ARMENIAN_FONT,
            font_size="15sp",
            color=(0.5, 0.5, 0.5, 1),
            size_hint_y=None,
            height=20,
        )
        self.add_text(self.subtitle_label)

        input_wrapper = NeumorphicInputField(size_hint_y=None, height=48, padding=[10, 0, 10, 0])
        self.url_input = TextInput(
            hint_text="YouTube URL",
            font_name=ARMENIAN_FONT,
            font_size="16sp",
            background_color=(0, 0, 0, 0),
            foreground_color=LIGHT_TEXT,
            hint_text_color=(0.5, 0.5, 0.5, 1),
            multiline=False,
            cursor_color=(0.7, 0.1, 0.2, 1),
            padding=[5, 12, 5, 5],
        )
        input_wrapper.add_widget(self.url_input)
        self.card.add_widget(self.title_label)
        self.card.add_widget(self.subtitle_label)
        self.card.add_widget(Widget(size_hint_y=None, height=10))
        self.card.add_widget(input_wrapper)
        self.input_wrapper = input_wrapper

        self.status_label = Label(
            text="Ready",
            font_name=ARMENIAN_FONT,
            font_size="13sp",
            color=STATUS_LIGHT_TEXT,
            size_hint_y=None,
            height=30,
        )
        self.add_text(self.status_label)

        self.submit_btn = NeumorphicButton(
            text="DOWNLOAD MP3",
            font_name=ARMENIAN_FONT,
            font_size="15sp",
            bold=True,
            color=LIGHT_TEXT,
            size_hint_y=None,
            height=48,
        )
        self.submit_btn.bind(on_press=self.start_download)

        self.footer_label = Label(
            text="The file will be saved in the app Downloads folder",
            font_name=ARMENIAN_FONT,
            font_size="13sp",
            color=(0.4, 0.4, 0.4, 1),
            size_hint_y=None,
            height=30,
        )
        self.add_text(self.footer_label)
        self.card.add_widget(self.status_label)
        self.card.add_widget(self.submit_btn)
        self.card.add_widget(self.footer_label)

    def add_text(self, widget):
        self.text_widgets.append(widget)

    def change_language(self, language):
        language_data = LANGUAGE_DATA[language]
        self.current_language = language
        self.language_button.text = f"{language_data['code']}  {language}"
        self.title_label.text = language_data["title"]
        self.subtitle_label.text = language_data["subtitle"]
        self.url_input.hint_text = language_data["url_hint"]
        self.status_label.text = language_data["ready"]
        self.submit_btn.text = language_data["download"]
        self.footer_label.text = language_data["footer"]

    def current_text(self, key):
        return LANGUAGE_DATA[self.current_language][key]

    def open_language_menu(self, *_):
        menu = BoxLayout(orientation="vertical", padding=dp(12), spacing=dp(8))
        popup = Popup(
            title="",
            content=menu,
            background="",
            background_color=(0, 0, 0, 0),
            size_hint=(None, None),
            size=(dp(230), dp(285)),
            separator_height=0,
            auto_dismiss=True,
        )
        for language, language_data in LANGUAGE_DATA.items():
            row = BoxLayout(size_hint_y=None, height=dp(42), spacing=dp(8))
            row.add_widget(FlagBadge(
                language_data["flag"],
                size_hint=(None, None),
                size=(dp(28), dp(18)),
            ))
            language_button = Button(
                text=f"{language_data['code']}   {language}",
                font_name=ARMENIAN_FONT,
                font_size="14sp",
                color=DARK_TEXT if self.dark_mode else LIGHT_TEXT,
                background_normal="",
                background_color=DARK_SURFACE if self.dark_mode else LIGHT_SURFACE,
                size_hint_y=1,
            )
            language_button.bind(on_press=lambda _, selected=language: self.select_language(selected, popup))
            row.add_widget(language_button)
            menu.add_widget(row)
        popup.open()

    def select_language(self, language, popup):
        self.change_language(language)
        popup.dismiss()

    def open_quality_menu(self, *_):
        menu = BoxLayout(orientation="vertical", padding=dp(12), spacing=dp(8))
        popup = Popup(
            title="",
            content=menu,
            background="",
            background_color=(0, 0, 0, 0),
            size_hint=(None, None),
            size=(dp(190), dp(270)),
            separator_height=0,
            auto_dismiss=True,
        )
        for quality in ("128 kbps", "160 kbps", "192 kbps", "224 kbps", "256 kbps"):
            quality_button = Button(
                text=quality,
                font_name=ARMENIAN_FONT,
                font_size="14sp",
                color=DARK_TEXT if self.dark_mode else LIGHT_TEXT,
                background_normal="",
                background_color=DARK_SURFACE if self.dark_mode else LIGHT_SURFACE,
                size_hint_y=None,
                height=dp(38),
            )
            quality_button.bind(on_press=lambda _, value=quality: self.select_quality(value, popup))
            menu.add_widget(quality_button)
        popup.open()

    def select_quality(self, quality, popup):
        self.quality_button.text = quality
        popup.dismiss()

    def open_settings(self, *_):
        content = SettingsPanel(self.dark_mode)
        title = Label(
            text="Settings",
            font_name=ARMENIAN_FONT,
            font_size="20sp",
            bold=True,
            color=DARK_TEXT if self.dark_mode else LIGHT_TEXT,
            size_hint_y=None,
            height=32,
        )
        mode_row = BoxLayout(size_hint_y=None, height=42, spacing=10)
        sun = Label(text="☀", font_name=SYMBOL_FONT, font_size="22sp", color=(0, 0, 0, 1), size_hint_x=None, width=32)
        mode_label = Label(
            text="Dark mode",
            font_name=ARMENIAN_FONT,
            font_size="14sp",
            color=DARK_TEXT if self.dark_mode else LIGHT_TEXT,
        )
        mode_button = NeumorphicButton(
            text="ON" if self.dark_mode else "OFF",
            font_name=ARMENIAN_FONT,
            color=DARK_TEXT if self.dark_mode else LIGHT_TEXT,
            size_hint=(None, None),
            size=(62, 32),
        )
        mode_button.dark_mode = self.dark_mode
        mode_button.surface_color = DARK_SURFACE if self.dark_mode else LIGHT_SURFACE
        mode_button.update_canvas()
        mode_row.add_widget(sun)
        mode_row.add_widget(mode_label)
        mode_row.add_widget(mode_button)

        quality_row = BoxLayout(size_hint_y=None, height=42, spacing=10)
        self.quality_label = Label(
            text="Quality",
            font_name=ARMENIAN_FONT,
            font_size="14sp",
            color=DARK_TEXT if self.dark_mode else LIGHT_TEXT,
            size_hint_x=None,
            width=70,
        )
        self.quality_button = NeumorphicButton(
            text="192 kbps",
            font_name=ARMENIAN_FONT,
            font_size="14sp",
            color=DARK_TEXT if self.dark_mode else LIGHT_TEXT,
        )
        self.quality_button.bind(on_press=self.open_quality_menu)
        quality_row.add_widget(self.quality_label)
        quality_row.add_widget(self.quality_button)

        content.add_widget(title)
        content.add_widget(mode_row)
        content.add_widget(quality_row)
        self.settings_popup = Popup(
            title="",
            content=content,
            background="",
            background_color=(0, 0, 0, 0),
            size_hint=(None, None),
            size=(dp(310), dp(230)),
            auto_dismiss=True,
            separator_height=0,
        )
        mode_button.bind(on_press=lambda *_: self.toggle_dark_mode(mode_button, mode_label, title, content))
        self.settings_popup.open()

    def toggle_dark_mode(self, button, mode_label, title, content):
        self.dark_mode = not self.dark_mode
        button.text = "ON" if self.dark_mode else "OFF"
        Window.clearcolor = DARK_BACKGROUND if self.dark_mode else BACKGROUND_COLOR
        self.card.dark_mode = self.dark_mode
        self.card.surface_color = DARK_SURFACE if self.dark_mode else LIGHT_SURFACE
        self.settings_button.dark_mode = self.dark_mode
        self.settings_button.surface_color = DARK_SURFACE if self.dark_mode else LIGHT_SURFACE
        self.settings_button.color = DARK_TEXT if self.dark_mode else LIGHT_TEXT
        self.card.update_canvas()
        self.settings_button.update_canvas()
        for themed_button in (self.language_button, self.submit_btn):
            themed_button.dark_mode = self.dark_mode
            themed_button.surface_color = DARK_SURFACE if self.dark_mode else LIGHT_SURFACE
            themed_button.color = DARK_TEXT if self.dark_mode else LIGHT_TEXT
            themed_button.update_canvas()
        for widget in self.text_widgets:
            widget.color = DARK_TEXT if self.dark_mode else LIGHT_TEXT
        self.status_label.color = DARK_TEXT if self.dark_mode else STATUS_LIGHT_TEXT
        self.url_input.foreground_color = DARK_TEXT if self.dark_mode else LIGHT_TEXT
        self.url_input.hint_text_color = (0.7, 0.7, 0.73, 1) if self.dark_mode else (0.5, 0.5, 0.5, 1)
        self.input_wrapper.dark_mode = self.dark_mode
        self.input_wrapper.surface_color = (0.20, 0.20, 0.24, 1) if self.dark_mode else (0.89, 0.90, 0.92, 1)
        self.input_wrapper.update_canvas()
        content.surface_color = DARK_SURFACE if self.dark_mode else LIGHT_SURFACE
        content.bg_color.rgb = content.surface_color[:3]
        mode_label.color = DARK_TEXT if self.dark_mode else LIGHT_TEXT
        title.color = DARK_TEXT if self.dark_mode else LIGHT_TEXT
        button.color = DARK_TEXT if self.dark_mode else LIGHT_TEXT
        button.dark_mode = self.dark_mode
        button.surface_color = DARK_SURFACE if self.dark_mode else LIGHT_SURFACE
        button.update_canvas()
        self.quality_button.dark_mode = self.dark_mode
        self.quality_button.surface_color = DARK_SURFACE if self.dark_mode else LIGHT_SURFACE
        self.quality_button.color = DARK_TEXT if self.dark_mode else LIGHT_TEXT
        self.quality_button.update_canvas()
        self.quality_label.color = DARK_TEXT if self.dark_mode else LIGHT_TEXT

    def start_download(self, *_):
        url = self.url_input.text.strip()
        if not url:
            self.set_status(self.current_text("empty"))
            return
        self.submit_btn.disabled = True
        self.set_status(self.current_text("downloading"))
        threading.Thread(target=self.download_mp3, args=(url,), daemon=True).start()

    def set_status(self, message):
        self.status_label.color = DARK_TEXT if self.dark_mode else STATUS_LIGHT_TEXT
        Clock.schedule_once(lambda *_: setattr(self.status_label, "text", message), 0)

    def download_mp3(self, url):
        download_folder = os.path.join(App.get_running_app().user_data_dir, "Downloads")
        os.makedirs(download_folder, exist_ok=True)
        quality = getattr(self, "quality_button", None)
        preferred_quality = quality.text.split()[0] if quality else "192"
        options = {
            "format": "bestaudio/best",
            "outtmpl": os.path.join(download_folder, "%(title)s.%(ext)s"),
            "noplaylist": True,
            # yt-dlp-ի warning-ները ուղարկում ենք Kivy-ի ճիշտ logger-ին։
            "logger": Logger,
            "postprocessors": [{
                "key": "FFmpegExtractAudio",
                "preferredcodec": "mp3",
                "preferredquality": preferred_quality,
            }],
        }
        try:
            with yt_dlp.YoutubeDL(options) as downloader:
                downloader.download([url])
            self.set_status(self.current_text("success"))
        except Exception as error:
            self.set_status(f"{self.current_text('error')}: {error}")
        finally:
            Clock.schedule_once(lambda *_: setattr(self.submit_btn, "disabled", False), 0)


class MainApp(App):
    def build(self):
        return DownloaderScreenUI()


if __name__ == "__main__":
    MainApp().run()
