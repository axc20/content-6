# Slicing

## String Slicing

```python
# indexing and slicing strings
spam = 'Hello, world!'
spam[0] # 'H'
spam[-1] # '!'
spam[0:5] # 'Hello'
spam[:5] # 'Hello'
spam[7:] # 'world!'

# string[start:end:step]
# end index is exclusive
text = "Hello World"
slice = text[1:4] # 'ell'

start_slice = text[:5] # 'Hello'
end_slice = text[6:] # 'World'

# negative indexing
slice = text[-5:] # 'World'

# step (defaults to 1) - how many items to skip
slice = text[0:11:2] # 'HloWrd'
```

## List Slicing

```python
# list[start:end:step]
my_list = [0, 1, 2, 3, 4, 5]
slice = my_list[1:4] # [1, 2, 3]

start_slice = my_list[:3] # [0, 1, 2]
end_slice = my_list[3:] # [3, 4, 5]

slice = my_list[-3:] # [3, 4, 5]

slice = my_list[0:6:2] # [0, 2, 4]
```
