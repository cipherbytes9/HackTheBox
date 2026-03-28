# Lucky Dice - HTB CTF Challenge

## Category
Misc

## Challenge Description
A dice game where you must correctly identify the winning player for 100 consecutive rounds within a tight timeout.

## Files Provided
- `Lucky_Dice.zip` - Password-protected zip file (password: `hackthebox`)
- Inside: `challenge.py` - The Python game server code

## Solution

### Step 1: Extract the zip
```bash
unzip -P hackthebox Lucky_Dice.zip
```

### Step 2: Analyze the challenge
The challenge runs a dice game where:
- 8-13 players roll 2-26 dice per round
- Player with highest sum wins
- On ties, the player who rolled last wins
- You must answer correctly for 100 rounds within 0.3 seconds each

### Step 3: Identify the vulnerability
The challenge expects you to actually calculate the scores. However, there's a simple socket-based approach:
1. Connect to the service
2. Parse the dice rolls from each round's output
3. Calculate the winner (highest sum, last player wins ties)
4. Send the player number as answer
5. Repeat for 100 rounds

### Step 4: Exploit
```python
# solve.py - connects, parses dice, calculates winner, sends answer
```

## Flag
[REDACTED]

## Time to Solve
~1 hour (debugging socket communication was the main challenge)
