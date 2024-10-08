# DjangoTrainee

- Before continuing please note that I have very less experience in django and solved this things with the help of django documentation, and my knowledge in python and flask. 

## Question 1 
Question 1: By default are django signals executed synchronously or asynchronously? Please support your answer with a code snippet that conclusively proves your stance. The code does not need to be elegant and production ready, we just need to understand your logic.

Answer => Django signals are executed synchronously by default means one thing happens after another. 


- So in this code I need to prove that one task is completed before the other and not at the same time.
- i will need to use signals ( post_save in it), receiver and User 

here is the basic code for this 

```py
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import User

# Signal handler (this will be executed when the signal is triggered)
@receiver(post_save, sender=User)
def simple_signal_receiver(sender, instance, **kwargs):
    print("Signal executed")

# Saving the user will trigger the signal
user = User.objects.create(username='testuser')
print("User saved")
```

so this will output the following thing 

```
Signal executed
User saved
```

- although this is not that clear, as things can be printed in order if they are asynchronously, so what we can do is import time and use it for the first function 

```python 
import time
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import User

# Signal handler
@receiver(post_save, sender=User)
def slow_signal_receiver(sender, instance, **kwargs):
    print("Signal started")
    time.sleep(5)  # Adds a time in it to slow the process. 
    print("Signal finished")

# Save the user
user = User.objects.create(username='testuser')

print("User saved")
```

So if Django signals are asynchronous what should happen is the statement "User saved" should appear first in the terminal as we have added more time, but this is what we get as a output 

```
Signal started
Signal finished
User saved
```

It proves that django signals are executed synchrously.


## Question 2

Question 2: Do Django signals run in the same thread as the caller?

Answer => Yes, Django signals run in the same thread as the caller by default.

It is hard to prove but I searched this on google and found that there is a module threading which you can use to know if they are using the same thread or not. 

- so what I can do here is take the previous code block and check the threads on both the signals and creating the user. 


```py 
import threading
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import User

# Signal handler
@receiver(post_save, sender=User)
def signal_receiver(sender, instance, **kwargs):
    print(f"Signal thread: {threading.get_ident()}") # check what thread is this signal using

# Save the user
user = User.objects.create(username='testuser')

print(f"Main thread: {threading.get_ident()}") # to check what thread is the main python code or main thread is 
```

And the thing is, yes. Both of the outputs in the terminal are the same. 

```
Signal thread: 140735984001467
Main thread: 140735984001467
```


## Question 3 

Answer => Yes, Django signals run in the same database transaction as the caller if they are triggered inside a transaction.

To show this, I can simply import the transaction module and then check if the Django Signal runs in the same database transaction.

```py
from django.db import transaction
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import User

# Signal handler
@receiver(post_save, sender=User)
def signal_receiver(sender, instance, **kwargs):
    # Check if this signal is running inside a transaction
    print("Signal running within transaction:", transaction.get_connection().in_atomic_block)

with transaction.atomic():
    user = User.objects.create(username='testuser')
    print("Within transaction block")
```

the output of the above code is simple, 

```
Signal running within transaction: True
Within transaction block
```

It shows that Django signals runs in the same database module. 


## Custom Classes in Python

Description: You are tasked with creating a Rectangle class with the following requirements:

1. An instance of the Rectangle class requires length:int and width:int to be initialized.
2. We can iterate over an instance of the Rectangle class
3. When an instance of the Rectangle class is iterated over, we first get its length in the format: {'length': <VALUE_OF_LENGTH>} followed by the width {width: <VALUE_OF_WIDTH>}


- here is the completed task 

```py
class Rectangle:
    def __init__(self, length: int, width: int): # Constructor for the rectangle 
        self.length = length
        self.width = width

    def __iter__(self):  # Method to make the object iterable
        yield {'length': self.length}
        yield {'width': self.width}

# Usage
rectangle = Rectangle(10, 5)
for dimension in rectangle:
    print(dimension)
```

- I often use function as compared to classes and it can simply be written as

```py
def rectangle(length, width):
    yield {'length': length}
    yield {'width': width}

# Usage
for dimension in rectangle(10, 5):
    print(dimension)
```


