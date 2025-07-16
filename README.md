# CatCkilckerInMyWindows-RU-
(Русская версия) Кликер созданный через нейросеть!! Это обычный .py файл (весь скрипт будет в README)
import pygame
import sys
import tkinter as tk
from tkinter import font as tkfont, messagebox
import random
import math
import time

# ================== НАСТРОЙКИ ==================
DEFAULT_SETTINGS = {
    "FACTS": [
        "Факт: эта игра сделана на deepseek",
        "Факт: мы специально сделали синий экран чтобы вы испугались :)",
        "Факт: клики дают очки!",
        "Факт: NewGeneration не знают, что такое роблокс",
        "Факт: чтобы запустилась игра, надо добавить разные расширения",
        "Факт: этот кот — потомок легендарного Ньян-Кэта!",
        "Факт: если нажать ESC, игра закроется (но вы и так знали)",
        "Факт: автокликеры — лучшие друзья ленивых геймеров",
        "Факт: RGB-котик одобряет ваши клики!",
        "Факт: МЕГА-КЛИК — это как обычный клик, но МЕГА!",
    ],
    "CHAT_COOLDOWN": 5,
    "CAT_SPEED": 2,
    "GRAVITY": 0.7,
    "JUMP_POWER": 15,
    "FULLSCREEN": True
}

current_settings = DEFAULT_SETTINGS.copy()
# ==============================================

def fake_bsod():
    try:
        root = tk.Tk()
        root.attributes("-fullscreen", True)
        root.configure(bg="#0000AA")
        
        error_font = tkfont.Font(family="Lucida Console", size=24, weight="bold")
        tk.Label(
            root,
            text=":( ТВОЙ ПК СЛОМАЛСЯ!",
            fg="white",
            bg="#0000AA",
            font=error_font
        ).pack(pady=100)

        joke_font = tkfont.Font(family="Lucida Console", size=14)
        tk.Label(
            root,
            text="Шутка! Нажми ESC для пропуска или жди игру...",
            fg="white",
            bg="#0000AA",
            font=joke_font
        ).pack(pady=20)

        percent = 0
        percent_label = tk.Label(
            root,
            text=f"Загрузка... {percent}%",
            fg="white",
            bg="#0000AA",
            font=joke_font
        )
        percent_label.pack(pady=20)

        def update_percent():
            nonlocal percent
            percent += random.randint(1, 5)
            if percent >= 100:
                root.destroy()
                return
            percent_label.config(text=f"Загрузка... {percent}%")
            root.after(200, update_percent)

        def close(event=None):
            root.destroy()

        root.bind("<Escape>", close)
        root.after(1000, update_percent)
        root.mainloop()
    except Exception as e:
        print(f"Ошибка в BSOD: {e}")

class Cat:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.jump_power = 0
        self.is_jumping = False
        self.color = (255, 0, 0)
        self.color_phase = 0
        self.last_chat_time = 0
        self.current_fact = ""
        self.chat_alpha = 0

    def jump(self):
        if not self.is_jumping:
            self.is_jumping = True
            self.jump_power = current_settings["JUMP_POWER"]

    def update(self):
        if self.is_jumping:
            self.y -= self.jump_power
            self.jump_power -= current_settings["GRAVITY"]
            if self.y >= 400:
                self.y = 400
                self.is_jumping = False
        
        self.color_phase = (self.color_phase + current_settings["CAT_SPEED"]) % 360
        r = int((math.sin(self.color_phase * 0.01745) * 127) + 128)
        g = int((math.sin((self.color_phase + 120) * 0.01745) * 127) + 128)
        b = int((math.sin((self.color_phase + 240) * 0.01745) * 127) + 128)
        self.color = (r, g, b)

        current_time = time.time()
        if current_time - self.last_chat_time > current_settings["CHAT_COOLDOWN"]:
            self.last_chat_time = current_time
            self.current_fact = random.choice(current_settings["FACTS"])
            self.chat_alpha = 0

        if self.chat_alpha < 255 and self.current_fact:
            self.chat_alpha = min(255, self.chat_alpha + 3)

    def draw(self, screen):
        pygame.draw.circle(screen, self.color, (self.x, self.y - 20), 20)
        pygame.draw.ellipse(screen, self.color, (self.x - 30, self.y - 10, 60, 40))
        pygame.draw.polygon(screen, self.color, [(self.x - 15, self.y - 40), (self.x - 5, self.y - 20), (self.x - 25, self.y - 20)])
        pygame.draw.polygon(screen, self.color, [(self.x + 15, self.y - 40), (self.x + 5, self.y - 20), (self.x + 25, self.y - 20)])
        pygame.draw.circle(screen, (255, 255, 255), (self.x - 10, self.y - 25), 5)
        pygame.draw.circle(screen, (255, 255, 255), (self.x + 10, self.y - 25), 5)
        pygame.draw.circle(screen, (0, 0, 0), (self.x - 10, self.y - 25), 2)
        pygame.draw.circle(screen, (0, 0, 0), (self.x + 10, self.y - 25), 2)

        if self.current_fact and self.chat_alpha > 0:
            chat_font = pygame.font.SysFont("Arial", 20)
            chat_text = chat_font.render(self.current_fact, True, (255, 255, 255))
            chat_surface = pygame.Surface((chat_text.get_width() + 20, chat_text.get_height() + 20), pygame.SRCALPHA)
            pygame.draw.rect(chat_surface, (30, 30, 30, self.chat_alpha), (0, 0, chat_surface.get_width(), chat_surface.get_height()), border_radius=10)
            chat_surface.blit(chat_text, (10, 10))
            screen.blit(chat_surface, (self.x - 150, self.y - 100))

def run_game():
    try:
        pygame.init()
        
        # Инициализация экрана
        if current_settings["FULLSCREEN"]:
            screen = pygame.display.set_mode((0, 0), pygame.FULLSCREEN)
        else:
            screen = pygame.display.set_mode((800, 600))
        
        pygame.display.set_caption("RGB Cat Clicker!")
        clock = pygame.time.Clock()

        # Получаем размеры экрана
        screen_width, screen_height = screen.get_size()
        
        score = 0
        click_power = 1
        auto_click = 0
        cat_power = 0
        font = pygame.font.SysFont("Arial", 24)
        big_font = pygame.font.SysFont("Arial", 36)

        upgrade_cost = {
            "click": 10,
            "auto": 50,
            "cat": 100,
            "mega_click": 500
        }

        # Позиция кота по центру экрана внизу
        cat = Cat(screen_width // 2, screen_height - 100)
        running = True

        def draw_button(x, y, width, height, color, text, cost=0):
            rect = pygame.Rect(x, y, width, height)
            pygame.draw.rect(screen, color, rect, 0, 10)
            text_surf = font.render(f"{text} {'(Цена: ' + str(cost) + ')' if cost else ''}", True, (255, 255, 255))
            screen.blit(text_surf, (x + 10, y + 10))
            return rect

        while running:
            screen.fill((0, 0, 0))

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    running = False
                elif event.type == pygame.KEYDOWN:
                    if event.key == pygame.K_ESCAPE:
                        running = False
                    elif event.key == pygame.K_SPACE:
                        score += click_power
                        cat.jump()
                    elif event.key == pygame.K_F11:
                        current_settings["FULLSCREEN"] = not current_settings["FULLSCREEN"]
                        if current_settings["FULLSCREEN"]:
                            screen = pygame.display.set_mode((0, 0), pygame.FULLSCREEN)
                        else:
                            screen = pygame.display.set_mode((800, 600))
                        screen_width, screen_height = screen.get_size()
                        cat.x = screen_width // 2
                        cat.y = screen_height - 100
                elif event.type == pygame.MOUSEBUTTONDOWN:
                    if event.button == 1:
                        score += click_power

            score += auto_click / 60
            cat.update()
            if cat.is_jumping:
                score += cat_power / 60

            score_text = big_font.render(f"Очки: {int(score)}", True, (255, 255, 255))
            screen.blit(score_text, (20, 20))
            cat.draw(screen)

            # Кнопки улучшений
            button_width = min(400, screen_width - 40)
            click_rect = draw_button(20, 100, button_width, 50, (0, 100, 0), f"Улучшить клик (+1)", upgrade_cost["click"])
            auto_rect = draw_button(20, 160, button_width, 50, (100, 0, 0), f"Автоклик (+1/сек)", upgrade_cost["auto"])
            cat_rect = draw_button(20, 220, button_width, 50, (100, 100, 0), f"Кот даёт (+1/сек в прыжке)", upgrade_cost["cat"])
            mega_rect = draw_button(20, 280, button_width, 50, (0, 0, 100), f"МЕГА-КЛИК (+10 за клик)", upgrade_cost["mega_click"])

            mouse_pos = pygame.mouse.get_pos()
            mouse_clicked = pygame.mouse.get_pressed()[0]

            if mouse_clicked:
                if click_rect.collidepoint(mouse_pos) and score >= upgrade_cost["click"]:
                    score -= upgrade_cost["click"]
                    click_power += 1
                    upgrade_cost["click"] = int(upgrade_cost["click"] * 1.5)
                
                if auto_rect.collidepoint(mouse_pos) and score >= upgrade_cost["auto"]:
                    score -= upgrade_cost["auto"]
                    auto_click += 1
                    upgrade_cost["auto"] = int(upgrade_cost["auto"] * 1.5)
                
                if cat_rect.collidepoint(mouse_pos) and score >= upgrade_cost["cat"]:
                    score -= upgrade_cost["cat"]
                    cat_power += 1
                    upgrade_cost["cat"] = int(upgrade_cost["cat"] * 1.5)
                
                if mega_rect.collidepoint(mouse_pos) and score >= upgrade_cost["mega_click"]:
                    score -= upgrade_cost["mega_click"]
                    click_power += 10
                    upgrade_cost["mega_click"] = int(upgrade_cost["mega_click"] * 2)

            # Инструкции
            instr_text = font.render("SPACE — клик / ESC — выход / F11 — полноэкранный режим", True, (255, 255, 255))
            screen.blit(instr_text, (20, screen_height - 50))

            pygame.display.flip()
            clock.tick(60)

        pygame.quit()
        sys.exit()
    except Exception as e:
        print(f"Ошибка в игре: {e}")
        input("Нажмите Enter для выхода...")

if __name__ == "__main__":
    fake_bsod()
    run_game()
