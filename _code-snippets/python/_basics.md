# Python

## while

```python
prompt = "\nTell me something, and I will repeat it back to you:"
prompt += "\nEnter 'quit' to end the program."

active = True
while active:
  message = input(prompt)
  if message == 'quit'
    active = False
  else:
    print(message)

prompt = "\nPlease enter the name of a city you have visited:"
prompt += "\n(Enter 'quit' when you are finished.)"

while True:
  city = input(prompt)
  if city == 'quit'
    break
  else:
    print(f"I'd love to go to {city.title()}!")

# -----
# continue ignores rest of loop and returns to beginning
current_number = 0
while current_number < 10
  current_number += 1
  if current_number % 2 == 0
    continue
  print(current_number)

# -----
unconfirmed_users = ['alice', 'brian', 'candace']
confirmed_users = []

while unconfirmed_users:
  current_user = unconfirmed_users.pop()
  print(f"Verifying user: {current_user.title()}")
  confirmed_users.append(current_user)

# -----
pets = ['dog', 'cat', 'dog', 'goldfish', 'cat', 'rabbit', 'cat']

while 'cat' in pets:
  pets.remove('cat')

# -----
responses = {}
polling_active = True

while polling_active:
  name = input("\nWhat is your name? ")
  response = input("Which mountain would you like to climb someday?")

  # store response in dictionary
  responses[name] = response

  # find out if anyone else is going to take the poll
  repeat = input("Would you like to let another person respond? (yes/no) " )
  if repeat == 'no'
    polling_active = False

for name, response in responses.items():
  print(f"{name} would like to climb {response}.")
```

## Reading and Writing Files

```python
from sys import argv

script, filename = argv

print(f"We're going to erase {filename}")
print(f"If you don't want that, hit CTRL-C (^C).")
print(f"If you do want that, hit RETURN.")

input("?")

print("Opening the file...")
# modes: w (write), r (read), a (append)
# modifiers: w+, r+, a+ --> will open file in both read and write mode
# and, depending on char use, position the file in different ways
# default for open() is r
target = open(filename, "w")

print("Truncating the file. Goodbye!")
target.truncate()

print("Now I'm going to ask you for three lines.")
lines1 = input("line 1: ")
lines2 = input("line 2: ")

print("I'm going to write these to the file.")

target.write(lines1)
target.write("\n")
target.write(lines2)
target.write("\n")

print("And finally, we close it.")
target.close()

# -----
txt = open(filename)

print(f"Here's your file {filename}:")
print(txt.read())

print("Type the filename again:")
file_again = input("> ")

txt_again = open(file_again)

print(txt_again.read())
```

```python
filename = 'files/pi_digits.txt'

# Open the file in read mode
with open(filename) as file_object:
  # contents = file_object.read()
  # print(contents.rstrip())
  # for line in file_object:
  #   print(line.rstrip())
  #
  # Read all lines from the file
  lines = file_object.readlines()

pi_string = ''

for line in lines:
  # Remove leading/trailing whitespaces (including newlines)
  # from each line and append to pi_string
  pi_string += line.strip()

print(pi_string)
print(len(pi_string))

# all text is interpreted as a string
# if you read a number and need to work with that value in numerical context,
# use int() or float() to convert it
```

## Parameters, Unpacking, Prompting

```python
from sys import argv # import argv from sys module

script, user_name = argv # unpack the argv (argument variable)
prompt = '> '

print(f"Hi {user_name}, I'm the {script} script.")
print(f"Do you like me {user_name}?")
likes = input(prompt)

print(f"Where do you live {user_name}?")
lives = input(prompt)

print(f"What kind of computer do you have?")
computer = input(prompt)

print(f"""
Alright, so you said {likes} about liking me.
You live in {lives}. Not sure where that is.
And you have a {computer} computer. Nice.
""")

# -----
print("How old are you?", end=" ")
age = int(input())
height = input("How tall are you? ")

print(f"You are {age + 2} old")
print(f"You are {height} tall")
```

## Tuples

```python
# list value is a mutable data type: can have values added, removed, or changed
# tuple data type, like strings, are immutable: cannot be changed
eggs = ('hello', 42, 0.5)

eggs[0] # 'hello'
eggs[1:3] # (42, 0.5)
len(eggs) # 3

# if only one value in tuple, indicate with trailing comma
print(type('hello',)) # <class 'tuple'>

# use tuples to convey that you don't intend for that sequence of values to change

tuple(['cat', 'dog', 5]) # ('cat', 'dog', 5)
list(['cat', 'dog', 5]) # ['cat', 'dog', 5]
list('hello') # ['h', 'e', 'l', 'l', 'o']
```
