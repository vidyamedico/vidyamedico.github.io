---
title: What You Should Know About The Django User Model
category: article
layout: post
published_date: "Jul 8, 2022"
short_content: "In this tutorial I’m going to show you how I usually start and organize a new Django project nowadays. I’ve tried many different configurations and ways to organize the project, but for the past 4 years or so this has been consistently my go-to setup."
featured_image: https://simpleisbetterthancomplex.com/media/2021/07/featured-user.jpg
---

![a]({{page.featured_image}})

The `goal` of this article is to discuss the caveats of the default Django user model implementation and also to give you some advice on how to address them. It is important to know the limitations of the current implementation so to avoid the most common pitfalls.
w
> Something to keep in mind is that the Django user model is heavily based on its initial implementation that is at least 16 years old. Because user and authentication is a core part of the majority of the web applications using Django, most of its quirks persisted on the subsequent releases so to maintain backward compatibility.

The good news is that Django offers many ways to override and customize its default implementation so to fit your application needs. But some of those changes must be done right at the beginning of the project, otherwise it would be too much of a hassle to change the database structure after your application is in production.

```python
from django.contrib.auth import get_user_model
from django.contrib.auth.backends import ModelBackend


class CaseInsensitiveModelBackend(ModelBackend):
    def authenticate(self, request, username=None, password=None, **kwargs):
        UserModel = get_user_model()
        if username is None:
            username = kwargs.get(UserModel.USERNAME_FIELD)
        try:
            case_insensitive_username_field = '{}__iexact'.format(UserModel.USERNAME_FIELD)
            user = UserModel._default_manager.get(**{case_insensitive_username_field: username})
        except UserModel.DoesNotExist:
            # Run the default password hasher once to reduce the timing
            # difference between an existing and a non-existing user (#20760).
            UserModel().set_password(password)
        else:
            if user.check_password(password) and self.user_can_authenticate(user):
                return user
```

**settings.py**

Please note that 'mysite.core.backends.CaseInsensitiveModelBackend' must be changed to the valid path, where you created the backends.py module.

It is important to have handled all conflicting users before changing the authentication backend because otherwise it could raise a 500 exception MultipleObjectsReturned.

##### Fixing the username validation to use accept ASCII letters only

Here we can borrow the built-in UsernameField and customize it to append the ASCIIUsernameValidator to the list of validators:  