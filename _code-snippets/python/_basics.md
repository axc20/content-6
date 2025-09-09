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
