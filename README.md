# Number-guessing-game
Python Number Guessing Game

import tkinter as tk
from tkinter import messagebox
import random

class NumberGuessingGame:
    def __init__(self, root):
        self.root = root
        self.root.title("Number Guessing Game")
        self.root.geometry("450x550")
        self.root.resizable(False, False)

        self.player_name = "Player"
        self.score = 0
        self.games_played = 0
        self.games_won = 0
        self.total_guesses = 0
        self.correct_guesses = 0
        self.leaderboard = {}

        self.init_ui()
        self.reset_game()

    def init_ui(self):
        tk.Label(self.root, text="Number Guessing Game", font=("Helvetica", 18, "bold")).pack(pady=10)

        name_frame = tk.Frame(self.root)
        name_frame.pack(pady=5)

        tk.Label(name_frame, text="Name: ").pack(side='left')
        self.name_entry = tk.Entry(name_frame)
        self.name_entry.insert(0, self.player_name)
        self.name_entry.pack(side='left')

        tk.Button(self.root, text="Instructions", command=self.show_instructions).pack(pady=5)

        tk.Label(self.root, text="Select Difficulty:", font=("Helvetica", 10)).pack()
        self.difficulty_var = tk.StringVar(value="Medium")
        for level in ["Easy", "Medium", "Hard", "Extreme"]:
            tk.Radiobutton(self.root, text=level, variable=self.difficulty_var, value=level).pack()

        tk.Button(self.root, text="Start Game", command=self.start_game).pack(pady=10)

        self.guess_entry = tk.Entry(self.root, font=("Helvetica", 14), state='disabled')
        self.guess_entry.pack(pady=10)

        self.submit_button = tk.Button(self.root, text="Submit Guess", command=self.check_guess, state='disabled')
        self.submit_button.pack(pady=5)

        self.feedback_label = tk.Label(self.root, text="", font=("Helvetica", 12))
        self.feedback_label.pack(pady=10)

        self.score_label = tk.Label(self.root, text="", font=("Helvetica", 10))
        self.score_label.pack()

        self.restart_button = tk.Button(self.root, text="Restart", command=self.reset_game, state='disabled')
        self.restart_button.pack(side='left', padx=30, pady=20)

        self.stats_button = tk.Button(self.root, text="Statistics", command=self.show_stats)
        self.stats_button.pack(side='left', padx=10)

        self.leaderboard_button = tk.Button(self.root, text="Leaderboard", command=self.show_leaderboard)
        self.leaderboard_button.pack(side='left', padx=10)

        self.quit_button = tk.Button(self.root, text="Exit", command=self.root.quit)
        self.quit_button.pack(side='right', padx=30)

    def show_instructions(self):
        messagebox.showinfo("Instructions",
            "1. Enter your name and choose a difficulty level.\n"
            "2. Try to guess the number between 1 and 100.\n"
            "3. You get fewer attempts on harder levels.\n"
            "4. Points are awarded based on remaining attempts.\n"
            "5. Track your wins and scores in the leaderboard!")

    def start_game(self):
        self.player_name = self.name_entry.get().strip() or "Player"
        self.target = random.randint(1, 100)
        self.difficulty = self.difficulty_var.get()
        self.attempts = {"Easy": 10, "Medium": 7, "Hard": 5, "Extreme": 3}[self.difficulty]
        self.original_attempts = self.attempts

        self.feedback_label.config(text="")
        self.score_label.config(text=f"Attempts left: {self.attempts}")
        self.guess_entry.config(state='normal')
        self.submit_button.config(state='normal')
        self.restart_button.config(state='normal')
        self.guess_entry.delete(0, tk.END)

    def check_guess(self):
        try:
            guess = int(self.guess_entry.get())
            if not 1 <= guess <= 100:
                raise ValueError
        except ValueError:
            self.feedback_label.config(text="Enter a number between 1 and 100.")
            return

        self.attempts -= 1
        self.total_guesses += 1
        self.guess_entry.delete(0, tk.END)

        if guess == self.target:
            self.feedback_label.config(text=f"🎉 Correct! The number was {self.target}.")
            self.correct_guesses += 1
            self.end_game(win=True)
        elif guess < self.target:
            self.feedback_label.config(text="Too low!")
        else:
            self.feedback_label.config(text="Too high!")

        if abs(guess - self.target) <= 5 and guess != self.target:
            self.feedback_label.config(text=self.feedback_label.cget("text") + " Very close!")

        self.score_label.config(text=f"Attempts left: {self.attempts}")

        if self.attempts == 0 and guess != self.target:
            self.end_game(win=False)

    def end_game(self, win):
        self.games_played += 1
        if win:
            points = self.attempts * 10
            self.score += points
            self.games_won += 1
            msg = f"Well done, {self.player_name}! You earned {points} points!"
        else:
            msg = f"You ran out of attempts. The number was {self.target}."

        self.leaderboard[self.player_name] = self.leaderboard.get(self.player_name, 0) + (self.attempts * 10 if win else 0)

        messagebox.showinfo("Game Over", msg)
        self.submit_button.config(state='disabled')
        self.guess_entry.config(state='disabled')

    def reset_game(self):
        self.feedback_label.config(text="")
        self.score_label.config(text="")
        self.guess_entry.config(state='disabled')
        self.submit_button.config(state='disabled')
        self.restart_button.config(state='disabled')
        self.guess_entry.delete(0, tk.END)

    def show_stats(self):
        accuracy = (self.correct_guesses / self.total_guesses * 100) if self.total_guesses else 0
        msg = (
            f"Name: {self.player_name}\n"
            f"Games Played: {self.games_played}\n"
            f"Games Won: {self.games_won}\n"
            f"Total Score: {self.score}\n"
            f"Guess Accuracy: {accuracy:.2f}%"
        )
        messagebox.showinfo("Statistics", msg)

    def show_leaderboard(self):
        if not self.leaderboard:
            messagebox.showinfo("Leaderboard", "No scores yet!")
            return

        sorted_scores = sorted(self.leaderboard.items(), key=lambda x: x[1], reverse=True)
        msg = "\n".join([f"{i+1}. {name}: {score} pts" for i, (name, score) in enumerate(sorted_scores)])
        messagebox.showinfo("Leaderboard", msg)

if __name__ == "__main__":
    root = tk.Tk()
    app = NumberGuessingGame(root)
    root.mainloop()

