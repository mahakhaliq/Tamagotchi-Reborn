# Tamagotchi Code Using Python
import pygame
import sys
import time
import random
from datetime import datetime

# Initialize Pygame
pygame.init()

# Game Constants
SCREEN_WIDTH = 800
SCREEN_HEIGHT = 600
FPS = 30

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
GREEN = (0, 255, 0)
RED = (255, 0, 0)
BLUE = (0, 0, 255)
YELLOW = (255, 255, 0)
GRAY = (128, 128, 128)
LIGHT_GRAY = (200, 200, 200)
DARK_GRAY = (64, 64, 64)
PINK = (255, 192, 203)
ORANGE = (255, 165, 0)

# Pet States
PET_HAPPY = "happy"
PET_SAD = "sad"
PET_SICK = "sick"
PET_SLEEPING = "sleeping"
PET_EVOLVED = "evolved"

class VirtualPet:
    def __init__(self):
        self.hunger = 80
        self.happiness = 80
        self.cleanliness = 80
        self.state = PET_HAPPY
        self.evolution_level = 1
        self.age = 0
        self.last_update = time.time()
        self.stat_decay_rate = 0.5
        self.is_sleeping = False
        self.sleep_start_hour = 22
        self.sleep_end_hour = 7

    def update_stats(self):
        current_time = time.time()
        time_passed = current_time - self.last_update
        current_hour = datetime.now().hour
        should_sleep = (current_hour >= self.sleep_start_hour or current_hour < self.sleep_end_hour)

        if should_sleep and not self.is_sleeping:
            self.is_sleeping = True
            self.state = PET_SLEEPING
        elif not should_sleep and self.is_sleeping:
            self.is_sleeping = False
            self.determine_state()

        decay_multiplier = 0.3 if self.is_sleeping else 1.0

        if time_passed > 1:
            self.hunger = max(0, self.hunger - (self.stat_decay_rate * decay_multiplier))
            self.happiness = max(0, self.happiness - (self.stat_decay_rate * decay_multiplier))
            self.cleanliness = max(0, self.cleanliness - (self.stat_decay_rate * decay_multiplier))
            self.age += time_passed / 60
            self.last_update = current_time

            if not self.is_sleeping:
                self.determine_state()

    def determine_state(self):
        avg_stats = (self.hunger + self.happiness + self.cleanliness) / 3
        if avg_stats < 20:
            self.state = PET_SICK
        elif avg_stats < 40:
            self.state = PET_SAD
        elif avg_stats > 85 and self.age > 10:
            if self.evolution_level == 1:
                self.state = PET_EVOLVED
                self.evolution_level = 2
            else:
                self.state = PET_HAPPY
        else:
            self.state = PET_HAPPY

    def feed(self):
        if not self.is_sleeping:
            self.hunger = min(100, self.hunger + 25)
            self.happiness = min(100, self.happiness + 5)

    def play(self):
        if not self.is_sleeping:
            self.happiness = min(100, self.happiness + 25)
            self.hunger = max(0, self.hunger - 5)

    def clean(self):
        if not self.is_sleeping:
            self.cleanliness = min(100, self.cleanliness + 25)
            self.happiness = min(100, self.happiness + 5)

class GameUI:
    def __init__(self, screen):
        self.screen = screen
        self.font = pygame.font.Font(None, 36)
        self.small_font = pygame.font.Font(None, 24)
        self.weather_types = ["sunny", "rainy", "cloudy"]
        self.current_weather = 0
        self.buttons = {
            "feed": pygame.Rect(50, 500, 120, 50),
            "play": pygame.Rect(200, 500, 120, 50),
            "clean": pygame.Rect(350, 500, 120, 50),
            "sleep": pygame.Rect(500, 500, 120, 50)
        }

    def draw_background(self):
        weather = self.weather_types[self.current_weather]
        if weather == "sunny":
            for y in range(SCREEN_HEIGHT):
                color_intensity = int(135 + (120 * y / SCREEN_HEIGHT))
                color = (135, 206, color_intensity)
                pygame.draw.line(self.screen, color, (0, y), (SCREEN_WIDTH, y))
            pygame.draw.circle(self.screen, YELLOW, (700, 100), 50)
            pygame.draw.circle(self.screen, ORANGE, (700, 100), 40)
        elif weather == "rainy":
            self.screen.fill((70, 70, 90))
            for _ in range(50):
                x = random.randint(0, SCREEN_WIDTH)
                y = random.randint(0, SCREEN_HEIGHT)
                pygame.draw.line(self.screen, LIGHT_GRAY, (x, y), (x - 5, y + 10), 2)
        else:
            self.screen.fill((150, 150, 170))
            for i in range(5):
                x = (i * 200) + random.randint(-50, 50)
                y = random.randint(50, 150)
                pygame.draw.ellipse(self.screen, WHITE, (x, y, 120, 60))
                pygame.draw.ellipse(self.screen, WHITE, (x + 30, y - 20, 80, 50))

    def draw_pet(self, pet):
        center_x, center_y = SCREEN_WIDTH // 2, SCREEN_HEIGHT // 2 - 50
        body_size = 80 if pet.evolution_level == 1 else 100
        body_color = self.get_pet_color(pet)
        pygame.draw.ellipse(self.screen, body_color, (center_x - body_size // 2, center_y - body_size // 2, body_size, body_size))

        eye_color = BLACK if pet.state != PET_SLEEPING else GRAY
        if pet.state == PET_SLEEPING:
            pygame.draw.line(self.screen, BLACK, (center_x - 20, center_y - 10), (center_x - 10, center_y - 10), 3)
            pygame.draw.line(self.screen, BLACK, (center_x + 10, center_y - 10), (center_x + 20, center_y - 10), 3)
        else:
            pygame.draw.circle(self.screen, WHITE, (center_x - 15, center_y - 10), 8)
            pygame.draw.circle(self.screen, WHITE, (center_x + 15, center_y - 10), 8)
            pygame.draw.circle(self.screen, eye_color, (center_x - 15, center_y - 10), 4)
            pygame.draw.circle(self.screen, eye_color, (center_x + 15, center_y - 10), 4)

        if pet.state in [PET_HAPPY, PET_EVOLVED]:
            pygame.draw.arc(self.screen, BLACK, (center_x - 15, center_y + 15, 30, 20), 3.14, 6.28, 3)
        elif pet.state == PET_SAD:
            pygame.draw.arc(self.screen, BLACK, (center_x - 15, center_y + 5, 30, 20), 0, 3.14, 3)
        elif pet.state == PET_SICK:
            pygame.draw.line(self.screen, BLACK, (center_x - 10, center_y + 10), (center_x + 10, center_y + 10), 2)

        if pet.evolution_level == 2:
            pygame.draw.ellipse(self.screen, LIGHT_GRAY, (center_x - 70, center_y - 20, 40, 20))
            pygame.draw.ellipse(self.screen, LIGHT_GRAY, (center_x + 30, center_y - 20, 40, 20))

    def get_pet_color(self, pet):
        if pet.state == PET_HAPPY:
            return GREEN
        elif pet.state == PET_SAD:
            return BLUE
        elif pet.state == PET_SICK:
            return RED
        elif pet.state == PET_SLEEPING:
            return GRAY
        elif pet.state == PET_EVOLVED:
            return PINK
        return GREEN

    def draw_stats(self, pet):
        pygame.draw.rect(self.screen, WHITE, (50, 50, 300, 150))
        pygame.draw.rect(self.screen, BLACK, (50, 50, 300, 150), 2)
        stats = [("Hunger", pet.hunger, RED), ("Happiness", pet.happiness, YELLOW), ("Cleanliness", pet.cleanliness, BLUE)]
        for i, (name, value, color) in enumerate(stats):
            y_pos = 70 + i * 40
            text = self.small_font.render(f"{name}:", True, BLACK)
            self.screen.blit(text, (60, y_pos))
            pygame.draw.rect(self.screen, LIGHT_GRAY, (150, y_pos, 180, 20))
            fill_width = int(180 * value / 100)
            pygame.draw.rect(self.screen, color, (150, y_pos, fill_width, 20))
            pygame.draw.rect(self.screen, BLACK, (150, y_pos, 180, 20), 2)
            value_text = self.small_font.render(f"{int(value)}", True, BLACK)
            self.screen.blit(value_text, (340, y_pos))
        info_text = [f"State: {pet.state.title()}", f"Evolution: Level {pet.evolution_level}", f"Age: {int(pet.age)} min"]
        for i, text in enumerate(info_text):
            rendered = self.small_font.render(text, True, BLACK)
            self.screen.blit(rendered, (400, 70 + i * 25))

    def draw_buttons(self):
        button_colors = {"feed": ORANGE, "play": GREEN, "clean": BLUE, "sleep": GRAY}
        for name, rect in self.buttons.items():
            pygame.draw.rect(self.screen, button_colors[name], rect)
            pygame.draw.rect(self.screen, BLACK, rect, 2)
            text = self.font.render(name.title(), True, BLACK)
            text_rect = text.get_rect(center=rect.center)
            self.screen.blit(text, text_rect)

    def draw_weather_info(self):
        weather_text = f"Weather: {self.weather_types[self.current_weather].title()}"
        text = self.small_font.render(weather_text, True, BLACK)
        pygame.draw.rect(self.screen, WHITE, (SCREEN_WIDTH - 200, 20, 180, 30))
        pygame.draw.rect(self.screen, BLACK, (SCREEN_WIDTH - 200, 20, 180, 30), 2)
        self.screen.blit(text, (SCREEN_WIDTH - 190, 30))
        instruction = "Press W to change weather"
        inst_text = self.small_font.render(instruction, True, BLACK)
        pygame.draw.rect(self.screen, WHITE, (SCREEN_WIDTH - 220, 60, 200, 25))
        self.screen.blit(inst_text, (SCREEN_WIDTH - 210, 65))

    def change_weather(self):
        self.current_weather = (self.current_weather + 1) % len(self.weather_types)

def handle_buttons(pet, ui, mouse_pos, mouse_clicked):
    if not mouse_clicked:
        return
    for button_name, button_rect in ui.buttons.items():
        if button_rect.collidepoint(mouse_pos):
            if button_name == "feed":
                pet.feed()
            elif button_name == "play":
                pet.play()
            elif button_name == "clean":
                pet.clean()
            elif button_name == "sleep":
                pet.is_sleeping = not pet.is_sleeping
                pet.state = PET_SLEEPING if pet.is_sleeping else pet.determine_state()

def main():
    screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
    pygame.display.set_caption("sagotchi Virtual Pet")
    clock = pygame.time.Clock()
    pet = VirtualPet()
    ui = GameUI(screen)
    running = True
    while running:
        mouse_clicked = False
        mouse_pos = pygame.mouse.get_pos()
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            elif event.type == pygame.MOUSEBUTTONDOWN:
                mouse_clicked = True
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_w:
                    ui.change_weather()
                elif event.key == pygame.K_ESCAPE:
                    running = False
        pet.update_stats()
        handle_buttons(pet, ui, mouse_pos, mouse_clicked)
        ui.draw_background()
        ui.draw_pet(pet)
        ui.draw_stats(pet)
        ui.draw_buttons()
        ui.draw_weather_info()
        if pet.is_sleeping:
            sleep_text = ui.font.render("Pet is Sleeping... Zzz", True, WHITE)
            text_rect = sleep_text.get_rect(center=(SCREEN_WIDTH // 2, 100))
            pygame.draw.rect(screen, BLACK, text_rect.inflate(20, 10))
            screen.blit(sleep_text, text_rect)
        pygame.display.flip()
        clock.tick(FPS)
    pygame.quit()
    sys.exit()

if __name__ == "__main__":
    main()
