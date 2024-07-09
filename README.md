# Mood-Tracker
Daily track your mood with more than 50 emotion descriptions and obtain a graphic

from datetime import date
import json
from matplotlib.dates import DateFormatter
import matplotlib.dates as mdates
from tkinter import messagebox
import matplotlib.pyplot as plt


MOOD_NAMES = {
    1: 'Happy',
    2: 'Sad',
    3: 'Surprised',
    4: 'Bad',
    5: 'Fearful',
    6: 'Angry',
    7: 'Disgusted'}


class MoodEntry:
    def __init__(self, mood_score, date, sub_mood=None, water_intake=0, exercise_time=0, sleep_time=0):
        self.mood_score = mood_score
        self.date = date
        self.sub_mood = sub_mood
        self.water_intake = water_intake
        self.exercise_time = exercise_time
        self.sleep_time = sleep_time

    def get_mood_type(self):
        mood_types = {
            1: {'name': 'Happy',
                'sub_moods': ['Playful', 'Content', 'Proud', 'Optimistic', 'Excited', 'Interested', 'Accepted',
                              'Powerful', 'Peaceful', 'Trusting']},
            2: {'name': 'Sad',
                'sub_moods': ['Lonely', 'Vulnerable', 'Despair', 'Guilty', 'Depressed', 'Hurt', 'Disappointed',
                              'Hopeless', 'Grief']},
            3: {'name': 'Surprised', 'sub_moods': ['Amazed', 'Confused', 'Stunned', 'Shocked', 'Startled', 'Excited']},
            4: {'name': 'Bad', 'sub_moods': ['Tired', 'Stressed', 'Bored', 'Sick', 'Busy']},
            5: {'name': 'Fearful',
                'sub_moods': ['Scared', 'Anxious', 'Insecure', 'Nervous', 'Weak', 'Rejected', 'Threatened']},
            6: {'name': 'Angry',
                'sub_moods': ['Let Down', 'Humiliated', 'Bitter', 'Mad', 'Aggressive', 'Frustrated', 'Distant',
                              'Critical', 'Annoyed', 'Rage']},
            7: {'name': 'Disgusted',
                'sub_moods': ['Disapproving', 'Disappointed', 'Awful', 'Repelled', 'Judgmental', 'Embarrassed',
                              'Nauseated', 'Hesitant', 'Horrified']}
        }
        while True:
            print('How do you feel today?')
            for key, value in mood_types.items():
                print(f"{key}. {value['name']}")

            try:
                mood = int(input('Enter the number corresponding to your mood: '))
                if mood in mood_types:
                    self.mood_score = mood
                    main_mood = mood_types[mood]['name']
                    sub_mood = self.get_sub_mood(mood_types[mood]['sub_moods'])
                    self.sub_mood = sub_mood
                    self.water_intake = self.get_water_intake()
                    self.exercise_time = self.get_exercise_time()
                    self.sleep_time = self.get_sleep_time()
                    return main_mood, sub_mood
                else:
                    messagebox.showerror('Invalid Input', 'Please choose a valid number between 1 and 7.')
            except ValueError:
                messagebox.showerror('Invalid Input', 'Please enter a number.')

    def get_sub_mood(self, sub_moods):
        while True:
            print(f"\nLet's narrow it:")
            for i, sub_mood in enumerate(sub_moods, 1):
                print(f'{i}. {sub_mood}')

            try:
                choice = int(input('Enter the number of your specific mood: '))
                if 1 <= choice <= len(sub_moods):
                    return sub_moods[choice - 1]
                else:
                    messagebox.showerror('Invalid Input', f'Please choose a number between 1 and {len(sub_moods)}.')
            except ValueError:
                messagebox.showerror('Invalid Input', 'Please enter a number.')

    def get_water_intake(self):
        while True:
            try:
                water_intake = float(input('Enter the amount of water you drank today (in liters): '))
                if water_intake < 0:
                    messagebox.showerror('Invalid Input', 'Please enter a non-negative number.')
                else:
                    return water_intake
            except ValueError:
                messagebox.showerror('Invalid Input', 'Please enter a number.')

    def get_exercise_time(self):
        while True:
            try:
                exercise_time = float(input('Enter the amount of time you exercised today (in minutes): '))
                if exercise_time < 0:
                    messagebox.showerror('Invalid Input', 'Please enter a non-negative number.')
                else:
                    return exercise_time
            except ValueError:
                messagebox.showerror('Invalid Input', 'Please enter a number.')

    def get_sleep_time(self):
        while True:
            try:
                sleep_time = float(input('Enter the amount of time you slept last night (in hours): '))
                if sleep_time < 0 or sleep_time > 24:
                    messagebox.showerror('Invalid Input', 'Please enter a number between 0 and 24.')
                else:
                    return sleep_time
            except ValueError:
                messagebox.showerror('Invalid Input', 'Please enter a number.')

    def __str__(self):
        return (f'I am feeling {self.sub_mood} today, {self.date}. '
                f'Water intake: {self.water_intake} liters, '
                f'Exercise time: {self.exercise_time} minutes, '
                f'Sleep time: {self.sleep_time} hours.')

    def to_dict(self):
        return {
            'mood_score': self.mood_score,
            'date': self.date.isoformat(),
            'sub_mood': self.sub_mood,
            'water_intake': self.water_intake,
            'exercise_time': self.exercise_time,
            'sleep_time': self.sleep_time
        }

    @classmethod
    def from_dict(cls, data):
        return cls(
            data['mood_score'],
            date.fromisoformat(data['date']),
            data.get('sub_mood'),
            data.get('water_intake', 0),
            data.get('exercise_time', 0),
            data.get('sleep_time', 0)
        )


class MoodTracker:
    def __init__(self):
        self.entries = []

    def add_entry(self, entry):
        self.entries.append(entry)

    def save_to_file(self, filename='mood_entries.json'):
        with open(filename, 'w') as f:
            json.dump([entry.to_dict() for entry in self.entries], f, indent=4)

    def load_from_file(self, filename='mood_entries.json'):
        try:
            with open(filename, 'r') as f:
                content = f.read()
                if content.strip():  # Check if the file is not empty
                    data = json.loads(content)
                    self.entries = [MoodEntry.from_dict(entry) for entry in data]
                else:
                    print('The file is empty. Starting with no entries.')
                    self.entries = []
        except FileNotFoundError:
            print('')
            self.entries = []
        except json.JSONDecodeError:
            print('Welcome to your emotion space')
            self.entries = []

    def get_chart_data(self):
        dates = [entry.date for entry in self.entries]
        mood_names_list = [MOOD_NAMES[entry.mood_score] for entry in self.entries]
        water_intakes = [entry.water_intake for entry in self.entries]
        exercise_times = [entry.exercise_time for entry in self.entries]
        sleep_times = [entry.sleep_time for entry in self.entries]
        return dates, mood_names_list, water_intakes, exercise_times, sleep_times


if __name__ == '__main__':
    tracker = MoodTracker()
    tracker.load_from_file()

while True:
    today = date.today()
    mood_entry = MoodEntry(0, today)
    main_mood, sub_mood = mood_entry.get_mood_type()
    print(mood_entry)

    tracker.add_entry(mood_entry)
    tracker.save_to_file()

    while True:
        choice = input('Do you want to add another entry? (y/n): ').lower()
        if choice in ['y', 'n']:
            break
        else:
            messagebox.showerror('Invalid Input', "Please enter 'y' or 'n'.")

    if choice != 'y':
        break

dates, mood_names_list, water_intakes, exercise_times, sleep_times = tracker.get_chart_data()

fig, ax1 = plt.subplots(figsize=(14, 6))

ax1.plot(dates, mood_names_list, marker='o', color='#F46A9B', label='Mood')
ax1.set_xlabel('DATE', fontsize=12, fontweight='bold')
ax1.set_ylabel('Mood', color='#F46A9B', fontsize=12, fontweight='bold')
ax1.tick_params(axis='y', labelcolor='#F46A9B')
ax1.set_ylim(0, 8)

unique_moods = list(set(mood_names_list))
ax1.set_yticks(range(len(unique_moods)))
ax1.set_yticklabels(unique_moods)

ax2 = ax1.twinx()
ax2.plot(dates, water_intakes, marker='s', color='#7EB0D5', label='Water Intake (L)')

ax2.plot(dates, exercise_times, marker='^', color='#FFB55A', label='Exercise Time (min)')

ax2.plot(dates, sleep_times, marker='D', color='#BD7EBE', label='Sleep Time (hrs)')

ax2.set_ylabel('Water (L) / Exercise (min) / Sleep (hrs)', color='#B2E061', fontsize=12, fontweight='bold')
ax2.tick_params(axis='y', labelcolor='#B2E061')

ax1.xaxis.set_major_formatter(DateFormatter("%Y-%m-%d"))
ax1.xaxis.set_major_locator(mdates.DayLocator(interval=1))
plt.gcf().autofmt_xdate()

lines1, labels1 = ax1.get_legend_handles_labels()
lines2, labels2 = ax2.get_legend_handles_labels()
ax1.legend(lines1 + lines2, labels1 + labels2, loc='upper right')

plt.title('Mood Tracker', fontsize=14, fontweight='bold')
plt.grid(True)
plt.tight_layout()
plt.show()
