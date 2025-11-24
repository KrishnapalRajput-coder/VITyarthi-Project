# VITyarthi-Project
from simulation.runner import SimulationRunner

# I usually just hardcode params while prototyping.
if __name__ == "__main__":
    sim = SimulationRunner(days=5)  # simulate 5 days first
    sim.start()
import random

class Environment:
    """
    Represents the outside world. 
    Not overthinking it — world is basically a mood influencer + random events.
    """

    def __init__(self):
        self.weather = None
        self.noise_level = 0

    def update(self):
        self.weather = random.choice(["sunny", "rainy", "cloudy", "stormy"])
        self.noise_level = random.randint(1, 10)

    def describe(self):
        return f"Weather: {self.weather}, Noise: {self.noise_level}"
import random

class RandomEvent:
    """Just a simple spontaneous thing that may affect the agent."""

    EVENT_LIST = [
        ("saw a dog", +3),
        ("lost keys temporarily", -4),
        ("got compliment from stranger", +5),
        ("coffee tasted awful", -2),
        ("found a cool rock", +1),
    ]

    @staticmethod
    def maybe_trigger():
        if random.random() < 0.15:  # 15% chance
            return random.choice(RandomEvent.EVENT_LIST)
        return None
class TimeManager:
    """Tracks current hour + rolls over into next day."""

    def __init__(self):
        self.hour = 7  # start in the morning
        self.day = 1

    def tick(self):
        self.hour += 1
        if self.hour >= 24:
            self.hour = 0
            self.day += 1
import random

class Traits:
    """Personality traits have subtle multipliers; nothing fancy."""

    TRAIT_POOL = {
        "introvert": {"social_need_mod": -0.4},
        "extrovert": {"social_need_mod": +0.4},
        "anxious": {"mood_fluctuation": 1.3},
        "stable": {"mood_fluctuation": 0.7},
        "lazy": {"energy_drop_rate": 1.2},
        "active": {"energy_drop_rate": 0.8},
    }

    def __init__(self):
        self.selected = random.sample(list(Traits.TRAIT_POOL), 2)

    def modifiers(self):
        mods = {}
        for t in self.selected:
            for k, v in Traits.TRAIT_POOL[t].items():
                mods[k] = mods.get(k, 1) * v
        return mods
import random

class EmotionState:
    def __init__(self):
        self.mood = random.randint(40, 60)  # baseline out of 100
        self.energy = random.randint(50, 70)
        self.social_need = random.randint(20, 50)

    def apply_modifiers(self, mods):
        # messy but human-like coding: apply only known mods
        if "mood_fluctuation" in mods:
            self.mood += random.randint(-8, 8) * mods["mood_fluctuation"]

        if "energy_drop_rate" in mods:
            self.energy -= random.randint(2, 5) * mods["energy_drop_rate"]

        if "social_need_mod" in mods:
            self.social_need += mods["social_need_mod"] * random.randint(-3, 3)

        # clamp
        self.mood = max(0, min(100, int(self.mood)))
        self.energy = max(0, min(100, int(self.energy)))
        self.social_need = max(0, min(100, int(self.social_need)))
class Memory:
    """Stores recent events; doesn't do anything deep yet."""

    def __init__(self):
        self.recent = []

    def add(self, event_description):
        self.recent.append(event_description)
        if len(self.recent) > 10:
            self.recent.pop(0)
import random

class DecisionMaker:

    ACTIONS = {
        "rest": lambda emo: emo.energy < 40,
        "go for walk": lambda emo: emo.mood < 50,
        "socialize": lambda emo: emo.social_need > 60,
        "work on hobby": lambda emo: emo.mood > 40 and emo.energy > 50,
        "make tea": lambda emo: True  # fallback
    }

    @staticmethod
    def choose_action(emotion_state):
        # try preferred first
        for action, cond in DecisionMaker.ACTIONS.items():
            if cond(emotion_state) and random.random() < 0.5:
                return action

        # fallback randomness
        return random.choice(list(DecisionMaker.ACTIONS.keys()))
from .traits import Traits
from .emotions import EmotionState
from .memory import Memory
from .decision_maker import DecisionMaker

class Agent:
    def __init__(self, name="Alex"):
        self.name = name
        self.traits = Traits()
        self.emotions = EmotionState()
        self.memory = Memory()
        self.mods = self.traits.modifiers()

    def tick(self):
        action = DecisionMaker.choose_action(self.emotions)
        self.memory.add(f"Did '{action}'")
        self.emotions.apply_modifiers(self.mods)
        return action

    def status(self):
        return {
            "mood": self.emotions.mood,
            "energy": self.emotions.energy,
            "social_need": self.emotions.social_need,
            "traits": self.traits.selected
        }
class Logger:
    def __init__(self):
        self.lines = []

    def log(self, line):
        print(line)
        self.lines.append(line)
from agent.agent import Agent
from world.environment import Environment
from world.events import RandomEvent
from world.time_manager import TimeManager
from .logger import Logger

class SimulationRunner:
    def __init__(self, days=3):
        self.agent = Agent()
        self.env = Environment()
        self.time = TimeManager()
        self.days = days
        self.log = Logger()

    def start(self):
        self.log.log(f"=== Simulation start for {self.agent.name} ===")
        self.log.log(f"Traits: {self.agent.traits.selected}")

        while self.time.day <= self.days:
            self.env.update()
            self.log.log(f"\nDay {self.time.day}, Hour {self.time.hour}")
            self.log.log(self.env.describe())

            # random event
            event = RandomEvent.maybe_trigger()
            if event:
                desc, mood_delta = event
                self.agent.memory.add(desc)
                self.agent.emotions.mood += mood_delta
                self.log.log(f"Random event: {desc} (mood {mood_delta:+})")

            action = self.agent.tick()

            status = self.agent.status()
            self.log.log(f"Action: {action}")
            self.log.log(f"Status: {status}")

            self.time.tick()
python3 main.py
