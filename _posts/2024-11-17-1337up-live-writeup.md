---
layout: post
title: "1337UP LIVE CTF - Funny writeup"
date: 2024-11-17 17:00:00 +0000
categories: security
tags: ctf
excerpt: The challenge is challenging, and you should try it
---

Another great CTF, this time from Intigriti. Playing in team `SavedByTheShell`, I solved 3/4 challenges in rev category. Unfortunately, I decided to only do Funny writeup. Buckle up and let's dive into the details.

### Description

464

`created by word_oh`

What is this funny file? 🤪

### Approach

We are given a fascinating binary: `funny.pyc`. Out of all executable formats, they decided to use `pyc`? What's the meaning of this?

> To speed up loading modules, Python caches the compiled version of each module in the `__pycache__` directory under the name module.version.pyc, where the version encodes the format of the compiled file; it generally contains the Python version number.
>
> ...
>
> A program doesn’t run any faster when it is read from a .pyc file than when it is read from a .py file; the only thing that’s faster about .pyc files is the speed with which they are loaded.
> 
> ~ [The Python Tutorial - 6.1.3. “Compiled” Python files](https://docs.python.org/3/tutorial/modules.html#compiled-python-files)

What can you do with a pyc then? Even without Python source code, you can still execute it in a similar fashion: `python funny.pyc`. If you fail to do so (_RuntimeError: Bad magic number in .pyc file_), you probably need python 3.13. Turns out, after every **minor** update in version, they renumber all the opcodes, so there is no way from one specific version of Python to execute pyc compiled using previous or later Python version.

<p align="center">
  <img src="data:image/jpg;base64,/9j/4AAQSkZJRgABAQEB9AH0AAD/2wBDAAMCAgICAgMCAgIDAwMDBAYEBAQEBAgGBgUGCQgKCgkICQkKDA8MCgsOCwkJDRENDg8QEBEQCgwSExIQEw8QEBD/2wBDAQMDAwQDBAgEBAgQCwkLEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBD/wAARCAECAPoDASIAAhEBAxEB/8QAHQAAAQQDAQEAAAAAAAAAAAAABgMEBQcAAggJAf/EAE4QAAEDAwMCBQIDAwkEBQsFAAECAwQABREGByESMQgTIkFRFGEycYEVI5EJFjNCUnKhscEYJGLRNEOCorIXJSc2N1Vjc5K14ThUdHXw/8QAGwEAAgMBAQEAAAAAAAAAAAAAAgMABAUBBgf/xAAtEQACAgEEAQIGAQQDAAAAAAAAAQIRAwQSITETBTIUIkFRYZEGFTNxgSOh0f/aAAwDAQACEQMRAD8A6UCwTgUolQHekW/xfpW9VBotWV8HYflX2oDJWI3OWbfaZc8DJjMOO4+ekE/6UOaS2c3U3A0JpfdX/aZt2mIGrmIb0W2vaQS+GnZOA1H80y09aipQSD0jJI45qZ1N/wCrF1//AIT/AP4DRDpPVugtA+CLa/W+5Cbuqz2OHpyalFrZ859cpL7P046O6kl3oyBjjPNNxpPsArS+bV+IG1ad3HvTO8don3rblp90WKJp0kXRAhoksEOqkAtFxKijHQoJWhQBUBmpa/bG66seqbVo7UPja0xYr/f0BdttUrSzDb8olXT0MIXPCnj1ZHpBP25qS2J3Um638YeunXtO3O2aa3C0rHkQIlzbSl5RtpQyVrQlRCA4mU5xknCE5weKq/dC/J174+tPagUCuPpTWdn0nByD6fKbW8+QD2JefcTn3CE+2KJ7Ugoxb6CzUeyuv9Na903thO8ZOnJOqNTvqSzaV6XZbnNRRHku/VJj/XFa2+qKpGcAZJ59OC/tXhq3mu+odU2F3xIRE/zQfjtuOfzNB+rDsVEjt9Z6MdfT3PbP2plu82g/yruz7hQCpOkwAcc48i+11napeln9RbhRrNa5Me7Rlxk3iQ4olEl0wkKaUgdRwEtFCT6U8g8HuReHHPtBwyZMPMHRxftbs3vJvJtrpXdLUO8Ni28jauQwuzW42Q3F2Ul5JU0VrL7QSpxIKwhPV6cZIOQM0n4fd8Na7p6y2mv25lq0zN0VDtVxauEWyGczeY00yQhwJU80WelUVaVD1erqwcAFVwbZ/wD6Y/Cpn50p/wDaXKsLTjzcfxZ7mPqHpb2/0kteO/E6+n/KlrR4OPlLX9T1fPzs4yn7e7n6P2Rt2++rdw461PXVFmm6XTY+gx3/AK9cNwfVeeSelSCr+iHxx3qX0Hs3rjeHQMzdmTvRZ9utKtXB6DEW/Zvr1vBp/wCnU44S80lsF4FCQOrOAeM1fPjsRa2fDEZFmLZhy9S2ic2pBylfnT0uqWP7xWT+tcw7O6V1VGYRvXrO03+97ENKfYVY4uspTEdqem6tt/VKtyXENOoS8h0qbVlK+vqIUaS9Ni8yW3iiytfqZaZyc+b/ANn3dzTWsfDrqxWktxb9Dv8AEkWpd2t14iRFRvqG21dLra2SpXQ4klJwFKGFpOckgTX+zXv5K231Xu3re6Q9AQtOWuTdYdkdiIuMye0wyt1XnKQ8lEfq6UgDK1cnqSnABIP5QW3sW3fXRNz1HqFyRadQ6cuMFi3yQ2lmGth1kuKQQASXQ+jPUSf3YwcYAmfDbMmzfAlvTGnXmfcWYitURoy5kpb6mmP2ehQQlSySEgqUQBxyfmlw0uHzyi4/+DMmv1PwkJqf1af3Bmx+DTdO7RLC3qTf7TmmtZahtb11i6TNjMgdDXleYkv+elR8svspWpLZCSvjqGCaWgR9W3XXFs2hnux7JqiXqpGk5rwR9S1FeDvQ48hOU+YkJBWkZT1DHIzXSPhT11r6LtLdvGh4rtePXWHa7VKh6UD9uiRHGba4tkvqQlhpvrXJfjx0NhWThpJTw5VGeGSTddzPGJorV+oGmW5t8vV71fNjp9SWiqI/5TaT79CnGwD/AMPzXc2mw7oJLlsDT67U7cjlK0l/2EXiT8Mu43he2ju28T+9TGsI1qkRmH7edNiD0JfdS0lYcElzstaOOnnPejid4TN6NDbgaI0xa9/GnoOtHJ7S7oNLdH7OfYimQ02WvqiHfMQh4Z6k9Pl9lZ4ujxf6fvutvCv4grTeLa4w1DYen2suIz58eHDiSQ4gf/NZeSD8proRJtMyXaYcroVOhtftCKknkYQWVLH5B4g/3xVtaXCnaiUJa3PONSk2ca2LaHeLUjGtnJnibt1ihbe3iRapsxejg4l9tmIxJXIUPrB5YAfIKefwZzzgONUWfcrYy6ada15ri1azsGpy7Hh3WLbjAdalJaLqULa8xwKSttKylQX/AFSCBxmw4P8A7OPFP/8A3l+/+wxKHvFxIjxdCbHPzB+4RqOMXB8p/ZErI/hmmOEYrhFfnJLkGtN7dbtbmaOsm5sjfOy7fWvU0ltqz25enjPcdS84URgtxUhoBxzghISfxDnviLvWptZ7Yalu+2mvmmL7qGCYSrW7bG/LF2blrLUfpbUf3a1OpU2UlRAUDyRg002M0rqKw6q05ubuDYtRXXbbUc6yNaIY/nrKfi2iW84pLDyraXUs9OVtYBSry+jKRmtPEm1Zrd4odV2HWN/XNRf9JWu5W8SVoZVCQl+U2lhlSAkkJcZU4Ccqy4eTgVyUUo2SKp0iT1Ttz4l7Ft3I3Rvt3sthuS7hCgW/Rv0yZalmXLZjM+fNQ70Nr63skIQ4AAPUSSAROeGvetq8OWuP4mNMydSJt4uY085phTbSmyooBLwklYbKwU+Z5RP/AA+1RLDeqdT/AMm1Z4NqvNwkagnTIMC2zX7g4mQmQdRNtRlfUE9aCk+XhecpCQR2ot8JGk9TaBvE/bzeCHfzuw/p96c7qaXqh6++da1TnUstsmStaWC2VoygNhKykKV1GiUI/Ym5or/RWsn9TacbuU6GqFPbddiTYx7syGllDiP0Ukin8i5gA8n+FV7tPFlWm033T064yLnNs2pLtbpM1xIC5jzMx1C31AcBS1JKjj3NGYYfc4LKifgDmqvEZMf2jV24uLyMGmT7jrisdOTipiPYLrKI+mt7zufZLZzUtD2/1K+Qr9mFAPuv00MuWSKoEWo7qyPT+dLfRr/s1YUbbeYwOqfNite+ArtS/wDMqzjhWpIoPuMiuA7WM1lDaCtRCQO57YqObv8AZnXxHbvERTnboD6SrP5ZoM32uNxg7bXCPa5TjE25OsW6O43+NLj7iWwR9/VVvb0+FPYiwbU69vOg9t7VbdXWnTE24wJ7S3Q41JSw6ppeeo/12j7VYjDcA5UDKtQWJl7yXb1CQsHHQp9IP8M06cvFoitpdlXOI0hYylS3kpB/Ik0x2h2H2J1H4e9rdx9VbKQdUX/VUOyN3GV1vecVyy2hyUrpJ4R1FauwwDyKe6N8LWycnxHbl7f3vSjd703b9P6cvNqtk6U883a3pTtybfQ0SvqCVfStrwScZwMAAUXi/JzeJzJFsvlnmxYdxjOofYW0VtuBQHUkjuPzqupusNZ33ZqyeHK46FssGwaf/ZTQ1KnU6XVupgyGnQTF+nHSV+VjHmnGe5xzWWsNWaUkRdWzNrdsZu2tstEM2x60yVrC5jwlqQZSQrkIUj0g/aut9XeDPaI7k6Audl0DDb006u5xNQ2xDjvkylORS7GfcBVnLbjCkjB/67nPtFF8pMFNXbKpn60u+2u5Fp3g0tp+3anehWeXZxAfvIt6Cl5xpfmB3ynO3lYx089XcY5BSi9J1Wd8HbRa0XleshrAWM3keV1BOPp/q/K/73l/pVkaU2A2WtWnd8d1NdaOd1Pa9EXy9sWbTzk11uHDhQWEu+W2kKwVrV1DrX1dICQAPV1NfEBtP4dPD5uXoOdN2qlX7Te4Li9PK06J7hhwpxfjeVNQHFkowhbiVIScH0kAHqKkvDlpLd0aOPUaeLbcG21XYGXzcDW+t/EbpjxRL0BZI150nbBa2NMN6pS8mYny5yPMMoRx0H/fs9PlK/o+/q4Nbf4tN0bDqTVN4mbFWBLmrX47jzK9dJSYYaioj9/o/Xno6uw749s1t4o9rditpJdg2+2x2OiQtWazjyn7dfIUhTYtgiOxy4pSVKJV1JewMfetfC1tbsTu5Kv+325uxkSbq3RcaI/cr7NkKcF0+rdkFtaUpUCnpSzg5+1AvKsnj3q++hzWmlgWbxOk67Brarf3dHZzbTSu2N3280juLD0mhhuzXBzUItz0VLSSloKT5DqVqbSShK09J6QAQTknXTfiR3c0fuXrbdK+6M07qmbreDbLb+z42oTAZssWF9SUNoUpl0vdRlLUonp9XUQMEBJl4eNtfC/vxqzcz6DYCDaIGipjFhZiSXluBUpp2X5shHSvgLAaGDz+7FIXLw37NW3aLYW9O7ZwI151TetLQr+44HUuykyIxVIbdBVx1KHqGAcimLHn4+dfoQ82jt3if7KjuG9uvdc7DaY8N+qNG2ro0+1aWV6mZ1EHXJIgrQUExfI4UpLaQf3p5yftT/Q+/mrdjdF3La6btjp3cLRsyc5PiQ592/Z7kVTjoeWhRLLqXUeb+8T6QQonkjABB4wNG7F7PXmFtptxsB+xb7d40K6MavhhwRYSRNAcjrJJHWtDSk4z/wBaKlfD/sdtVrDYS9b57hbZv7kX5V0mNJtPnukxY0eT5HlssoOCvoSp4+krV1BI9hSdmfz8SXX2LHk0vwtvG+/v+AB1V4l9Q673GO5W63h40JrC0M2j9lWfT1wuLTzVpUp0OOyvPeiuBx1fSlJKW2/ShI9iSH7U776r2h8PeqvD7btvrfqRrVLNxS9fTqQRlRlS4qWCfI+nX19HT1f0gz2471Zmndr/AAr7l+MPSej9mpUDUmgnrHdbhqXT7Up9yLElx0hpBKXCFoClPt+gHAUjOBzRN4tfDRsroVzZu8bZbe260wLvunZ9NX9mJ1huZCkPqbcbdyo5HU10cf2zTPHqO9y/Qh5tJxFQdf5AXV/jZs+vdGwNt9WeDPTV30xZ/IES2r10UxmQyjoawhMIDCUnAHYfpVa7R7yq2R3cm7z6Z2xtc+PIh3CDD00jUH07VrbkvsuJCJBYX1hCWigDy05C88Ywe39e+Bnw/Oal2/OldqbJAgI1I4q/sNBzEyD+zJ3S0v1dvqPplf8AYqH1X4SvDZbt+NutI2/aCwtWa92XUMmdFSlzy33GDB8lR9XdPmuY/vGuyw5pNPcuPwBDPpoJrY+fyc/teO3ca52ncaw3naJF+g7gSZK2WHNbEiyRHrezFVFZ6oigpAU049wEDqeV6c5Up/M8fO5M/czTO4EHZa2RkadstzssqznV/mftBEtyI4hzzPpB5Sm1Qxx0L6g4oenuQrxdR9ndDbgzdqtqtindDztMzA7J1AhKkxrwyuCHPKa6u/Sp5GeTy2aNt0dmtqtK+AnQ+9Ni0NboetLnbdMPzLy31+e6uQWPPUSVYyvqVnj3NA5Z1KUdy4X2GqOl2wk4P5nXY3snjq1LpT+fMW5+He0XiLr68v3aZCf1mG0tIeiMRlxzmGfMSUsEk4TnrIxxkxW7Pii1Z4gpOlIl/wBCWjRNl0zKXPahs3n692TILSmUHr8poIQltbg6Qk5Ku/GKufwe+GTYjcbw/aS11uvtnZ75qHUcy4rTMl9alrbEuR5KeFAYDLQ/hUV4dPCftDre/wC/ujtbaJiynNO65nWqwyXAtLtsgvR0OsJZ54CEupKSc9geaYoZpRW6S5/Al5NNCT2xfH5Ndnt3d0rJoezbcaf2f05ubZ7BKRKs0h6+mA/C8t0uNeYksPBSmlHCFjpwkJ4JGSpqvSW7e6131rP15sntxfr7rC2JtNouSrkwqVo9hDTqEeQpyOtx1aXHlPFSVtZX2CeMLveGywaL0D4d9LW2E5pXVeq73b7JrG62x9bcqYGrNNfmNlXUQC45FPqAyCcjtVgal2U8Lzd/1Tt41GiaBv8AarZCkwdQJ1I4xNU7IS90rT5rv7zoLI6uvrC+rkfLlGaVNlWUoN2kVu3A3r0/sdF8Mlt0lp2AzaHWZEXV7mq2itp9m4pmocMMsckLSE4Lv3+1HD/iB3ehzzeJ20+2DF//AGeLWvVf87SpoNhfVj6byOvy/MPV5Xn9/wCt71C6P2A2j1H4dtpNdan0JAnam1KrS6b1cFvOqcmKkOsJkFRC8EL6lZx80yX4adopXjlXta3oC3/zDibWt3x+z9bn06ri5dHWUvEdWeotpUO+PSK7UvucuP2JrbqRsVoHTKY101XG1FeXnnp9xloWFefLdWXHnCBx6lqUakHfETtZFeLGn9NMSHhwOhCSSfyoO318Ne1lj3L2T09tbppnS9h1/cptrvC7b1JLzQYRKRhSirpWW2H0pI7dZPtRze/D/wCEm26k1Dt01YLToO82m1QJsC/C/LYmKckGQlK0+a5+86DHBV19YX14I45X4W+2Esi+wGat8RWpZsQx9N2mPbXm8k4UkKV9sHtTGyXLfPV0RMuZeY1raWPQp+QlAXn45p3A0Dslpjwi6Z8QG4GxaNwNSuWy3LuKYC3VSZsh95DKnUkHGAV9ZwPwg0NbO/7LNx24tl93Mu2n9Ua01PqZqBA0fN1WgS7TEl3cRYzDUQOBWWmXUuqygqV0n1BOMc8H5O+X8FiwvDzuTfGkSrtroFDvPWw+VJ+fyqQ/2VXzyvW8wq9/X7/xpnetFT9sd+TtRoPW12tGlr5YBfYluW4qSIL6X1NOttLdUVBsgJUEknpJUBxgAmO3WpVEqO5l1yeT+6RSJxlB0HF7lZVWuIovmuNrdMIQ6r9oa4tTy0tjJLcZ4SF5+3S0c/bNddBiyXbcrV9if1RHmSJmmbW1KsJjnriR1PXBKZClk4Ul4qcQE4GPpycnq44Z1C7eL09CuFuv86z3O1uqfg3CEvofjLUhSFKQcHGUrUPyJqEat+57F6k6lib662bvs6KxBl3FMwee/HYW4tppR6eUpU+6R/fNWoTUVTFODbOsdpZeoNt/Cts1aoMvyZkd3TVgmKUyk9bTkpqO+npVnpJSVc9wfepTbjTVm074wt4ZNoh+Qu+aS0jc5p61KDkgv3dkrAJ9PoZbGBgcE9ya5Dc0/u3Lhx4UjfbXDsViQ1MjxzKT0svtuB1Dg9PCkrAUPvSSrHvLFu0zUUHezXzV7nxo8KXcUSR5r8dhTq2WlHpwUoVIeIH/AMRVF5Yrs5sYP726j3nuydRTfEBY5lr1HJZcg2ZLkJqO29a2ZvUlSA2T1Y81OVHB9Vemp1fAY13C0FIWgTJ1ldu8VOeVNsPNtPH9DIZ/+o158Nbd6u1eQ7ujrXUOp30R3I8aRdnwtcdtakqUEYAxkoSf0FTcnQ+vpl7i6olb464cvdvYkQ4VxM4edGjvqbU60k4/CostEg+6E/FVZ63Fhbtl/TemZ9XFPGi6rVCl6p2R8TmltOR13G7ydR6uhNQo/rdW+7DR5bYSOepXUnA98igj+USdbc1fsDb0uJMhGrUSVNdXrDYkQ0FWPjqWBn5NBNu2svNjuNw1HpjdTWNl1Bel+ZdrpCuKkO3FzOet9P4FqHOCRkZOMZNbSNq0XCS9edTarv8Af9QO+T5d5uU1T8pjylBbQaKuGwlY6glIAzk4yTVeXquFrhM0cf8AHNUpXJqv8lneMjP/AJe9oCP/AHPqL/xwKzwbZO/m75P/ALo07/459V5F0tf13iNftabgag1bNgIeagu3Z8OmKh0pLiUYAx1eWjP90V8laYv7d6k33Rm4F/0lNuCGWpztpfDRlIaKy2leQc9PmLx/eNVfjsXxXm+lUaH9Iz/074bjddhX/J1dbWovEEscFOs3VDI9/NlVYuudR3XWO0vhz1ffXEOXK+at0jcZim0BCVPPRluLISOw6lHj2qgIm3t90e1Nn6J3S1Rppy6PfU3NVvlBBuD2SS47kcqJUo5+5oEusHW0G32qxjerWTtv065Hds8VUsdEByOnpYU2McFCeBV2PqeFL6mVP+P6ty4r9l6+Pm770qvsPT0eyynNoRbYVwvU9mE0tLNxbnEthTpPWkeljgAg5+9QfhHtG4GltrtUb07a6pu98Kr3JZuG3wiNrivvtvIR5rK8ea08Y6kLJSSFYHUlWBiir3qXdO+QXrJrLfHWGoLPLKfqbdOlJUy8ErCkhQA5wpIP6VExr7qnSNwl3LbzXWo9JPXDH1f7InKabkEDAUtvlBUAMBRGccZpfx+Hzb1dUNl6PqY6bxSpO7/z/s9IY+0OjrV4p293bFZIsS8T9HTbfeHmEhBfUqZEUw44lPClkNvJ6zyQ2BnCaDdydNWnVeyWn37DqhjUybLuvarq3PQ2W0oeRq1IksgZVyz5j7Oc8ls9s4HBsHX+5Vtvs/UULe3XUW8XZthq4TzclOOyGmestNnqBCUpLrhCU4AK1H3p9bb1uJarT+w9M746th2synbgYTMpPlmU5IVJcdwR+IvqU4T/AGjmrS12Kr5KT9G1H4/Z6cX/AF9HsF4161JICNLaZiagWe5CXBOB4/KGf40Jag6z4htl1KJJ/mtqTJPz022vOa53jda5vXmRdN69Yynr/bW7NdFuSxmZBQXihhfHKB9S/j/5qq1l623qduUK9O766yXcrYy9HgyjLT5kZp7o8xCD08BXlN5/uihfqOJdp/oi9G1L+37LF8f123wnbpXONuBYZEfbW0ThG0bcBDbQ2/JftyFvILoPWs9Tb+AocdB+1W5ubofW+4f8mTt3pnb3TEy/3pentKvtQogT5ikNoYUtQ6iBwkE965R1DfNy9dxI1t3J3V1Jq2DDkiXHi3OQFttPhC2w4AAPV0OLH5KNP7XrffLT9thWHTm/Os7VaLdHbiQoEaWkMx2EJCUNoHTwkJAA+wpMdbh3ybvkfL0vUeOEVVpnoJs5pmDonYvw+2K96iY0/MgN20iKWisXKc9apPmRQQR0kqedd6iDy1jHORK2hlvazVm/uvXYx8l8w9TlI58xLNobaVgfcxFDH/OvNq46i3hvLdpbvG9mr5iNPzGrhaUuSwBCktIUhtxsAAApStYH50rP17vpc2JsO679azmR7nGMOc25LTiQwQoFtfp5Thaxj/iNOXqGF8KyvL0fUrl1+z0W38vVpOsvDxeUz2BBmbipLD/WOhzz7BdUtYPY9aloA+SoVJXXZ3brcXdTVszc/YzSGomYsG2LtV7vmn4011wKQ8l2Oh15CiENltCukHgvk45ry+vEvWmp7HadKaw3L1LeLJp51h+0QZEw9EB1hstsuNEYUlSEKISc5Ga6x8Ltg13u63LN+8Re5SZ1oCQ0hN3JQtheMhQI9R9A9RyeTzycuhqoT6K+XQ5cMd0i4toWHNReFXZdGnIhlogyNN+cmKnqDKIsptL+QPwhvy19X9npPxRhp2yRpXiz15q1K2XFQtCabs4Uk5U0tU26vuIV8ZSY6sfBB+KG4fhNtWmI6YG2+6eudGxCvrkRbVc1JYec/rOeWrKUqV7qSAT71pE8KDVhlS7hpLeHXlllXWQmXeJDNyK3rm8E9IcfWsFSiEgJHOAOBim+RFRRbFbhZ7BK0JsJf4WrmNQW7S2pLYWb4EFlEwOwZNvQ4ElRI63JLYwSeVDmiO8bQbd7kbs6mkbn7F6P1HHhWu1qtd7ven4011wrMkPRkOPIUQlsttr6QeC+T70Cs+DayN6bZ0I7uhrR7RsZLYasC5+IrZbdDzak8dQUlxKVgg5CgCOacSvD5qqPLbfi+JDc8COkoabVdypKUnGQcj1H0jlWT355NTyImxkPY7tu1pjwHaSuOw9nMzVse22tuDEYhtv/ALgy0Jf6WlkJPSwXCO2McVS20WiPD1pjYW27wbx7Uage1ppnV0iVOu1vs0ky2psTUTiWkeaB5B6VtobWkK/CFJHqq6ofhx1XYoUS1aZ8Q+4lntUBtLMeBFnoDTSB2SAU8CnWm/DOiI2u1TN1NX3CwrmO3GRZpMsKiyZTskynHVpxypT6lOHn8RJqeRHVBsYRbncN4t6kbuxdKXex2O1WAWWB+1WgzIlrW6p1xzygT0IGUpGTk4JwOKsUsyM/i/woudgstNBtIwEpCR+VMzGaz2pTk27HRqKo4TOs7NDdXFmtvNPNqKVIKD7GncbVcd89Vts8uU7/AFQlo12IrYHbgzTdn7E08+4cnqHBJ+RRPA0TpS0oQiJYYrXT2KEDNCdtHF9sk7oXZaWrNot4dRCepbRBTn3OanjtzvvLYL78ZMRlPKipXSAK7LYjRo4/cR0IGPYDNBW5l++jtzduQfVJwVAHkChye1jcMd+RR7soO1265wobca6TfqXkZClBOAaeFlBFPiB1H35NZ5A+1eezS8j4PfaPBDBiUUNUNJIrR5pIwadKQEn0/wCFauM9Sc59qR42XiJktBXGcYpuhsJJOTT99gk8GtmofUj8GTk+1RQ5ICt/muvMlhlCkkDj781XNytFwkuLCQoqPtV0TbdHQnzH0AJx3IoNu93s1uUpSUICh+WTRpWJborqPoaSrqdnp6AfUDTC46VjNpUGljqPFTN/142/1NtnoSMgc4yKE3tUoUsFSwRn3NMhH5irmdqgbvmmJ0Vlx9AKgnkY70Gx75IsrwPrSATkHNXVDnMXRjo6EHPBoX1XoKHNQt6CxjAz3yc1bqik1RAwNXw7gnpW5hXAp4paHcKSTg8iq9uNrlWSYcIc4PweKe2zUslWG1EEDjk8igyK0C210GqfxCnQOABgUONXpPGVc/nT+NdgtecdSQKVtOpt8snG2EL7nFR89AaUOnnvS7FybHdSf4io65zUr5GPeihGpWdyT+UZPveokqxmr88IWtXNN7r21lyb5MS4BUZ1JPCs9q5zckeoqPI+9EWhLq/H1TaHWHFtqTMaIUk4I9Qq9p+zO1VSxtM9kAElIPB98188tPuTimdhdU/ZYLqiSVR2ySfc9Ipy68lnJWcY9jWizyze10NZ8huO2cY5z3qDU99SonAAPxSl2lecSArgZ7GmEInqwc/lXDm4XcQAjue9bwU+UFLHOeK0lHGAOOacxAPK7DvUJuEH184wKb4HwKXkfiP50hUJuDAMNLScUmqEg4xW8QZTkmg3VG8GlNLyzDkqcdcScLDYyR/jUOQhKbqKDFuG2nggHNUlu5JaXqERmurDKOk5HvVn6f3B05qmEubZ5yFdI5aUcLT+YrnXVGvImodwbpbGlOqXEcKF9acDj4NV9TNxjwa3puDJDNuaNm20q4Pelw0k+nFJMZyOrueadgAcgVgfU93jraqI9xrpVwaRWoDKf0p297mo91SvM4Pc0RYEzjrORUrb47SopfUQAknvUapvpCnFdgKhb7rFu12l1lpWFEnp49650QGtzNbfRvGJGWAlvhRB71RF91hJmKUQtXJIFPtc35cyStHmErdyc/epDbzQkKb5N91FcG40AHrBUM5+1SPYpoD4ml9RajBeihQRnJUrIB+1KTtu79CZWt5XUQnIxng1bWu929s9B2xpFtaUorUQXfKPRkdsH3NUre/Edpm4PmO1JWFLVn+jOKfD7lDNNR7BSReNT6WfWf3npPA9sVL2bfNoAJucHqcBxlOMf402vGp7XfYvUl1Cgr3AqqL/AB0R3FLbGBnNWopNWU1kUmdAO3jSetAlcVxkOnko4BoJv+nVWmVhlI6VkqBA7VUMG/SrfMbejOFK0qBATx2q+tO3E6706lxKD9Y16Ck9zxUlFUMjFS7BFDEpawhKgSo4FFw05cLPaU3Cc8hIeGQjByKldEaDlXbULDEttSGW15WR9qm9/wCZFs8BqJG4SyQkHGM8VWx/NKmHKCjHgrdu5pJ4X2pR2UXEYCs5oFgX1h10JK8EnuTwKL7eY0loBEhvqP8Ax4/zq0sO3kqymmqQm6pYIST70RaK41Ha/tLaP/eFQM2OplWCoK6TyQc1K6UmtRr/AG15xXShElsqP2zTsKplTP7Gey2myE6ct6yePpmz/wB0U3ucwEnpJ4pnZLk2dL2x1leUriNKH5dIpvJcLhyfer7PK5PexlJdUVHCu9KQM5GaavgBXFPbekEA1wAVldx+dOYn9D+tNpf+tL29RU3g/wBqoQ+OMkrKieAa08tPxTl0DqwPmtOgf2agykfdaXxvTekLneXHOgMR1kEfOOK4n0jcZWuZr97YeWtvzlB0qOSCT710x4lrm5B2iuqWhkup6Sc4x2rjPwu3l9OorlbFr/3eQkqKO4BB7ik557Y19T2P8a0qnjllkrLJfd1Noq6i7WGW4hR4WjHoWPv9qj9BibPu9xvc4D6iW+px0jgZJq17j9FLX9GptGCOkVEs6aEAqcjIAQTk4FYuTNLcbGXTJy4VDppQScn3pzTMcDHxTlKiTikXbsbCDjSNXz6MVHeRlfVxnNSD/wCGm6EdQJz2oiyRV3kCPHUnPq7VU+rLgfKUjJ6u9WRqErKVJBz3qr7/AA1yFqSc4oWQqu6wH3pilttlQVR5pLTLyjbZd3Ut61R3R5reeMnsSPenkaxoVgLZB++KPrJdLNa7W5BnxkraUjBHzxUK2T2ldeLjR2lL1oQOxLwzbBb2lzY3SgBMhaQMtn7kcCvPkIWtxwpH9b5rujdkR77ZTYkuqXFWsqwo56T7fwrmKbt+INxWQR5YXx0juKv44JrkyMybsb2GFLatZlBZRxjFRVymfUPFhR5Pej51lpuEmO2gJT09PFAV4tMlp9TjIU4VdsJNNSpUV4xbXBCmKpT4CQe47V0v4edLSnAuSvIbWMVUuldFXG6Ijy0MdYUoJI6e3z3/ACrtDaLSLdoszALGCoJJJHzS8vCNDTY93ZM2HT0eypcnODHlgkHGa5s8QUx6b9WVjCVKJH5V2DcoX/m51ptA9QwTXJe/9leaZfcQCcdWeKHE7ZY1cahwcjuXtcSYtLLhAHuPalIuqbi275jc11JHtmoOe1ic6Os8E18isKWvpSs5PatjH7DyuTLLfRYcDXVyd6EyZLjgGMZNGFi1C4qSzMJwUKCh+YOf9KqiDZ5yvU0hxRHOMUW2IvtdLL6VIWBntQyVBRycNyParYXVH88NptOXcqysxEtL4xynijt9ASOr4qnPB+HBsLp1bndSFnH61csop8snPOKNdGBm4m2yKeUCsAVJ2v0jJ+9RvlhbncjFS0JsIQOSeKgsSm8nNOrOkqBA/tU2lDPFOrP6FYHzUILSgUrII96QqRnNp7+9MfLHzUIVf4nmlubUzgEnvngVyb4b9LuxGp2qZJQPOJbZHwArk13Du9aDf9CXGC1wryie2fauGtVa0f29tqdNW5CUeSjClDglR71U1XB9A/iuVeFwLguep7PbpDapFwjBSiDhTqc4ops1zh3KAZEdYUhSeDnIP5Vw6/rG7ahu7EJ15avNX5YIPI+9dl6G09IsOk4EOUVeaGElXV84rEk7Zv54K0x860nrJSABmvgznilHVBKcU2deS2DzmpQlRo2cUMYKuaTWClrKRjjmm7T3nuce1N77MdishLaVDIxmo2GlZCXxec4Vk5oOlsJWo9QGfvTu63l0LUFk0NvXlSnCQo1EyNUSzbWEfiwaRmRA7HVlzB+M1EqvfR3XTeTfgUY66keytKMpKmDuoLOtSlqSsnuara/2V7qUelXf4q1ZNxaeSQSScc/ehm6x/qVBKEnlXNXMeTjkzsmKVtFUqtcpSihtHWM9iMmi/TekPrAlTrAPykp70Y2PRaZTgV5Hf3Paj+0aUj29HU42Dj3FG8qoDHhcXRBaS0XEhqbaTDbQjOenpGM1eemYDLEJLCUpHTgAUAQGfLlAIHFWPp9KlJTxjIqtOaa4Zq4MNLkkJkVCmFAEDiucd6NPmbCkBScZ6gSR7V0rICW0KKzwBVI7qBDrD6R6gsdv1NTDkalRNViXjPNTWMBVtu8qMhJ6m3VA4Ht7VAMXF+K8Fpxkdwau/cbREV67OyG0qCnVEqJ+arSdotTLhU2UnPfNbWLI3GjxmbG4zsmdF64Ql0tTW0/2QaM45Tdrg22wgdbpwgJ7qP2qpW7W9Fcw22QQeSK6A8MGlRf9zdONXguJhomIeeWfZtBBP+VE3YuTqJ60+H2wq01s7pe0vw1R3moDSnG1j1BZSCcj2PNGlyUpHCVY/KgpzejR1pZRGblKdQykJ6kI4OBQ9c/EDpNa8NNuL+4TimLoxcr3SLMZUOpOT3qcYSAwlWAAR3qjEeIXTTJSfpnf+0aVd8S9nx0RobigOw6xUALkfwTgHNLW1QDg9WOaodfiWjhZxAI/WtGvEh1vgtwkdPyTUIdLvNeYgEHJIpmYxzVBSPEpcSgIb+jRxwoqAIqFV4lrmFEftGF3/tioQ6TnsCTBkMHHrbUOfyrzd3w0ndrlqq5G3wXXEpX6insCK6LleMa2oziZbmRjkOPoApXS8+0auYcvbIZeFwJc6m+UkH71U1SbVnqv4zqFim4M4k0PZZ1v17azdIDqGg+jPWMDGRXf762nYjam846BjJ+1VJuTtM3NKLpaWglba85SOatCCH27HDZkHLiGEhRPfIGKw32z2csiycoipClDP51ES1FKuD3qVkEFSkjvmoqXyrIrh1JH2AlRd9J796fXiB9RHHWMjp9uKZ2z+nH50QzRmERkZIzUCKJ1i0YbygDxkgVXr9xKFK9XOasXcEf70f8AhSQfzzVQ3RwpWoD5rtMlIWkXZauCqmrk9SsYWf40wW4VkD2p0wznBI/hRRVM41xwKsuuK56j9qmLcyHHAt0ZJphHiEqyBxnjmpZlPkIGO9FwV5YtwSRH0RUhCRjjPFTsF92QA2kqUo9h80F295Xnd6O9OoDqkgjgn4oH3R1YUiYs1olOvgpjnuM1YFtty4aElxOMjtX20IiwIiVSUjkA5707XcYsjKWF+kjAGDUpjYP6EXelqDCgDVJ6+C1JcWFe2P8AOrsnsJWyrPxVVa2gJKVHHtR4k9wvPzGjmrUFpbnOLQpGVKJqv7rpXpdUnygRz3q471ELcpxShgHtQvPbQlRKvvWnCVHnM+BSZWLWkWFL9bXY/NHUHWLe09sa1BCgh5xjgAqxjI75pRLLK/UDUJuTEbl6UktJOSlOf1ANOjK3SM3U4vHATunjK1M+SiNaWhjgZJNC9z8U+vpowy6zFz8DmqRk+l0pHGOKRq+sfFnnJdlvO+JDchWOm8pH/Y//ABTZ3xAblPkqN/Wnq/s8f6VViVYzmlEqUcY7V3b+DhYyt7txF99Ty/8A6q1O8uv3U+rU8z4/HVdvuBKeO+a0YcWTkVzakQsB/dXWUkFEi/z1A+5dNMjr2+k5N5l8/wDxT/zoUyT3rKlIg+cuVxcPUqQ8QO4U4o/616W+A7dOLq3RA0rOlo/aNlQEkE+pTZ4SfvTveLwM7IaFjoTZbJLV5sVLwdckEk8c8Cuc7LcLb4bdW2bW2nEPtQ1yjGuDS1lQWxg8foaVOFxNLQZ/Bkv7npgtJCSkdu+CM0NT9TxkXX9kBaVPFPbPYUN6V39291ZbETrReW5Lq0BQYbOV9WM4I7ihqEqddtZuakdHlNFKkoaA5A+9edzx2zPf6bIpQVBy+75jhUk8GkFNhRyTWvOcJz+lLIZJVjJpb6La7PkZIaeSoU8uslxuN1DnKe2ftSbbBTz0EmkrqFqhEHOefzxQKNBt0UrrqQtct3P3qq7mshah3q2NZRFecsqSeT3xVY3SGorVhB/hTY9ETsh2U+YoCp2JBUUpOSc01gW1RKT0nP5UX221KUE+kiutWdGEeCoJBArZxhY4I7UWxrAtSQQk9vikZtjWhOek/wAKBraC1YN2/Ifwas/RkUvqQCn3/wBKBWLWpt7PSR98Vau3EVtaQVjPQeSByOK4nyc2k3dvPYQlAJAKajLfJeS6ATnChW2u9Z6fsmDKnMoSlJ5Urmq9hb06AdkoYbv8QOuLCEgvAeonA/xqwoNqwYvaW7Pmx40bqdVgkcj4qsNW3+1LStHmJ4981P6uuqBakPIWCC3kqB4xiuata6ndZddSl1XzwqihFxdis0+B9qFTT61KZWlQJPvQPeMo5I+ahBrZ5mUouqWQOwJ705kX5q4MeZkZA7Zq5KD22Ys2kmRq5wQsp6sYPaonVlySqwykqI/Aff7UjOnJ89fSRyfahTW19QzblwwRlacd6LTxe6zN1s/+NoqWVnzFKI9zTdK8nGKcSyCpWDkA4FNkg9QODitlO0eWl2b0q3zx8VokpGerFbhSfY1G6OCT6snGKxhWFYxXxwgKOfitWDlRwc80NbuSD6srB2rKm1kPYzdfeYSo2j0XuxNS275aIxW6lfSpKl+k49uDXJPi/wBER7DZL3b47xcTaJTLiQBgYcA/51a27E3/ANG2015Sf3gtvlqJ+W3KgPFnFFyY110DJetNtnJz8KQkk/50AabbKX8CrEiduJLkFwlpmGolJ5AJVgGu/Gm0p7YGPtiuKf5P62FN71NK6eERm0D7c12oSQCRWF6hFRnwe/8AQ/nxrcPmVgKx3p2yOo5FRcJxS3SlWMZ+KnIzKPvWZ5VVG4l9RRBzhPxWs1tJbwUntThtlHV2NZNADZwOwoouySK11TYvqMuBIIz2qu5unASoeX71c9w9WQoAj8qiDYkTQS0j1E4o7aOx6Kpt+nlIWOpI70Z2iwBXTlAzU+jTKmyPRRDabMWsdafjFG3SOkbCsCAnBb7itZenm+kkozRzGgtoAJSOBTW5BlKVYSKFfN2DKaj2VJfLYmKwVNDpPJ/hSdg1uLXBdQlYac6FZP6VOandQtBBTxgiqg1RYpvStyG+CMH0pVyc0SimwN5Q2/eqbvfLk7GbXIc/EQUq7/lVYaZ2R3X1U+1ItNrkoaWoKD6lYSj3z3zV53DTcpyaVS4gWAe5HNWXo7WBssBNtMZpJIz2GR7Vp4oRURMsm3sZbUW3dWw2R3SuvXEXGKhoqZkFfqbwPwn7VVm5E9q2l9597KQo9zUnr3du/W++PCNIWhKSR0gcYqntVailat8xcpR9ZwUgYFH42+ilqNRXQCXrWb5kkxyfxEg5PFS2l9UXGWolbhKSMY+KYN6XL7oaQ0FFasCjROi4en4La21dTyhlRB4H2xTJrbEyFJtjYySpC3lnpx3zVe6jnqmyXEk5CTx+VF2pJSYUFbZOFL/D+VV1KePJT3UcnNWMCVWZmtyySoj3DkkfetQMkCtykE5rOkJBI71cXJitI0UnprBWFY/rVop1KVcUMuhZj34jWsXufzr4pwLz819i9z+dSPRCQwMZ6q+Um2c5z7UpREPRzcNaZPh320uCTkR3JsdR+MLyBTXfAInR7qVZ/wB+0JEeVn/hSB/pVuz9nEHw7WvTGo71HiPWK9yG1uDCsqOCByR81UusZFs1Fd52nbfcG5K7bpF2E6tJBCktoJ7j71WlJUFGO50C38n7G6rRqKWE8LkpaSfnAGa60mNhJ4+a5x8BcBtnbq5PpTlf7TWknHf0iuk5zYPOe9YHqLTyNI+j+iwUcEaEbctPndPvRNBAIwaFYSktPkk80XWxrzEglRFZaizYfY8QyAPzptPQryzgexqR8sgADmmc8EJKT8U1JvoCXQIT1BKufapTT7QdV1fHNRd0QOojPftRHpqIppgLUOVjsaYuABzKjsteoDn8qyG11LyRTuTHUsinkCKlpJU4OT24rtgSnStDaS10IJ+3FCF3fKevJ7UZXVxCQrHwarXU14isrW2VjqzjvRRF777BHUk8ZV0njFV/dLooqIPaiDUE8EKUD3+9Ad0fX1k1bwRbFyZGXWUHCSo9s4qLYkhKwc8Z70tMWV5z71H4OMA4PtVyCa7EyfBVe5j3Xdnz7YNArChwef4VZGtrLNlPOuIbCuOSKCk2h9vALZyBViPRk6lUSdkKUuIXgZBBozlIZmxelxAUAnP5YoHtzDzLg6gQnPJ+KKnJiWLS86VcpQcVzJHeyvCeyJT24Uppc8tMnhJxj4oHd9RyDxUpqaYZdwdWV9lkfnUOAfc5q7hiox4MHU5N8zCoDvWHlPFfCnJzmsCuenFG2ik+xJfYimx96dOp6f1pqoYJFAcEsn5pVhQSMk+9I0synqGM+9Qg/Q7kj4pTrTTdsAEAntS3o/tU36EPTrxW3K4QdMatgRpbqUt6kivABZAw4yc/5Cqi8Nxtc7VNyizFuLemWmVHQnq7qU2qrq8SOmrpqO2a3YgM+Y4H7TNSn36SkpJ/xqm9gNudZxdy7Q43Dwhbqml9+epJTj/GqklYeP3FkeCNj6Tbu5tLPrXdnwUk8pIwOa6ElNqUCMGqs8MuhLxou1X+zXyOlqQ1eJC+DkEFXGP4VcMxkJSTkCsHXr5tx9H9H/sRIEI6HTkYopsckEBKlY/Wh1woCznmn1te+nWkKUDzniqC5NeXAaJAwAO9MbikdKuMd6xualX7wK4IrSU8l39aYlQpysHXoSXpKEqwR1Z5oygRENMoAAAAqGjxUKfSfiiVASlCQkYGKJ9gOVGFhI9SsHHzSb6w0ACO/anClI6eTxVd6+1gbU4PJdOQrGBRbSvKXA+1hfo9tila30JJSeeoccVzVrvcuC1PU2JzZUD3Uf8AlS+vtUz76oNKdWCcnIVVO3rQk27yS6ZY9Xyk0SVAp2EUrdaAQUPPJc+6TxQ1d9yGVBSmXG1nHAAoUuW3smIpSVPqOP7ORUYrScsenrV/jV3THSUmbjylKISpLf2wDn+NRb+vJnVn64pyeUgYqGuWk5eSpKzkZx3ofmWW7tZT0KUB7jNX9pWn3QUXPXr7UZTgkgqV6T1DuKDXtwJiVnp8kjPcpps7ZLs+gpUhZxzzUDcbPOiglxo/kK6lRQ1UbC2JrttwdDiBlVSF01GXLDJKFnK8Ywe3eqvaRIS4CUlIB71NXF+SxbG2lZ6XjjPtxRRVujKy5NnAMzCFuKWecnNN6WkEHkUgoZGKtRe1UYuX3WZkfIpM9yawjBxWhWAcYqCWvqY4ScZNIL7knilHHBxxSSz1jioAI0o0SFd6TpVlJJzUIOxWZHyKwnA/KkC4M9qLcQ9txb0ahZcu8pAai323MMTErGFR3Wl5BAPfNBcqwx9HXxjUVj1AEphueYEdAyrH/wCah9wNzV2wO262y0j2Kge2Kqy1StR60vRt6ZDqmlkF1R7BOaVJxiuRuGDnNJHRm2XnSIs+5PulxUyUp0qz3JOc0S3dzy2gfvUVo6Em02pqE2vqDSE+r5OOay8zASoBR7/Fea1k98z6h6bh8WBEc7MSFkHvTpqTnBzzQ466ouE/605jSwlY6lVURcfYZRJ5KelZ4Han4fCgP8KEVS3GkpWkZTjk0/h3QOIBC/anJi5IK4ix1hSqlzKwABjtQxCnBYSMjiptpQdSCCKjEy7Mul1EZhawew96503Bvrs2a6rzPwKJ4+avHVT7ceC6tasDpzXNmqnfOmPLByCciiV2IIJgvXCQCkgqzjtU81YnGWy+tA4FRWnkkyM/8Qo0nLS3HClnHSjNGQq/Ua4zS1Dyxn34oZW7HXx5ac/lT7VMpciU6E98/PtQw++tltSlAjANXNOcbpWKXJURltTjjYxzQVdb1bULISnAzUbqDUMlKVs5/Dn3oClzpDjilkHKj81dtmflz1IOXL3A5CR2FQVxuEKVlsIBVQ557x7gj9aXjNlz1EGii+CpPJvYuYEZeQUD5oi3b0vHsOi9MSPJW2/cY6pDgIwO+Bj+FJaetip12gw0oKzIfbaCR3VkgYq0fGVa27TadFQWGehuLBDWM5wQBx/nTIe4ydSzk14YSKQUSBkUu/wOabqUCOKsmPKVs1JJ70i8oo7UoVBIyabPHqJI7VBbZqlZV3xxWy+E5FJoUBnNfHD1D01ATVRIGRS0UkjJ+aZOLSB05rdlzCOPmoQkVupAIpLzEf8A+NN/NJ4NZ1p+ahDtq56qvOqr+m06db+ulyXelPQMhIPuftXUG3mj06WszUWR0rluNj6hQ59XuAaHtp9nrZt1D+seSX7q8MuvnHGfYfAq0YcbzXE44HuPmsPU6xfQ9r6T6YuJZETtow3EKu+Bioi8LHUojn7VNNJDLIbTwnHP3obvC8OHCuM1jOdttnr40opL6EI+o9RUKQRIyoerFfZK8ZPVTFDyOrsKJAPsJI0wOs+QTye3NMXZcm3vqCvwdXHPtTBqb5TqTnH3p/PdYnQcqcQHAOCVDNOQEiXt+ogOnKu/3ortl5DwA8zj86pxctURSU5JAPf2qctWolNqADhwPvUFONh9qnzZULpbPUFJIqib/a3/AKhxJBGOO1XRGvbMljynFpOeBk0MagsiH3lutpGFcggU0RKO10VbaITkV7PJwod6KLnHcch+lOSUYxW37HUw4VdOMH4qahsMOM9Dys4Hb3qAlMzNNKkSVlaAST3PemFw0olLSklHUCPcYFXBeGIcFBdaaT+o5qu9R3ZI6kJIAPtVvB8tHGrKc1Ft4h5S3W0D3JFAU7Q8plwhLY6Qaue53JASoBQ5+9C0qShwqJxz96umTng2ysndNrayVIOB9q0j20p9PRjB+KN5pbKSCUmopLKCvOAKhVmtissjww6CVqnc9l9cTzEWSOq4KTjISEqABP6kU+8dFvLcWwPBPpC3Uk/9rirG8GtzhWWTq2S55X1Ey3ohNAkdeCvqOPc/hFCPjpR5umbSsd0yCM0+HuMbUySs4VlZBIIx3pmMe5p9MSepXT6h8io5ak4xkZqyZjjbsx3B9INNl8AiseUoK9KjikyvPdX+NQFqj5WqlYyK2yPkUm+eMiocEHAlRyDSzTQCBzTUEBQzxTxpQ6AMjNQhhRgE57VrSqvwn8qSqEPaaU6lJA5/SpO1J9PUQcZocRLQ9JKSfwnii21pR5Q6a8M+XbPrUUodDxfDZPsRxQveElSuPaieQo9qG7mCpZAo4qx0ZMFZ/UAcEVEpdIVk1N3NopSSKgFjCunNGiCrjx757femsi4ltOCeBSTrigCM1Dz5hTlJxxTTjVjx26hxYSc/ansN9xJz7UMQ1rkSUpTzk80cxLQ75KVIBII96NIXLh0PrfclIWkKziiyPJZkRh1rGccUAuOCGrpc4NInUYi5y76fzNGJmr5Cm7BpvrIUOM4qCRPDKiQoVETdWwpCelt8lXvzUFPviVJw05zmiirESmok7qO7Jcjn1c4xVT6in5cI6v8AGpy53jrYKVue9V3qW4LAUoqHA/1qxDigfLEhrzdOhSvV80Nv31GSMnjvTC8XFTi1ev5odfkuAEpPBq3GTZm5skbsJHbulz+tSf7ST/aoTS84TjqNLsvOFXQVHHempJmdlzpxLg2yvdwtNzaukFxSFNqByFEZ59x7irN3ZjDerTLbCJCWJcFSnVtp56/kiqY0Vd4zCAy4pKST3NWdpy7N2q4xrk08jpC8rTn8ST3ooz2uzLyrerZzrqTY7UMQKVayiWkf1RwrFVberLcLTIVFnRHGXUHBCk9q751hY/ImouMVxJhzgHmujsPkfxoG1TozT+pbetF0gIWvHDoGFp4+aZ50Ijg3dHFDxwnHf8qb1a+t9m7nZlGZZ0Llw8HhIytP/OqvlxXI6ggpWlefUlacEUyORMTmwuHKEK+rT1IGCK0UVJxkDmkluq6jTCuaOIJV1cYpaOepQHxSK1np7ClYiiT1H5qEHShwRWnQfkVuonBNadZ+BUIeuen54kyFYX1DPfNWHaXR0AdZ/jXO2m9bu26c42be7IQDypKscdHVxwcngj88fnRpG3ZlNpMxmzkw2GkrdwoqX1EnI7ADAHOexP258ZHBkatI+py1eJdsud5acZBqFlNl11XHAFQqtc/7rGfFuUrz0Ok/vfSChWMhWOUY9ZVgYQQrBzikrTq1+7zm46rFJjNPJBDjh5BIWcEY4/o1Dv7p+eOxxzSuhsNRjb2pi9yjfu+1CM1HlLyO49qN7j/RfrQTdiQo4+9Rdj30Qk+WWm1E8frQxOn9XuOfvT2+yVhBA+9AFwuj7bqxk4Cj71YgrYmUq7DexzWkzWwpYGSKthUsR7OHU46iOAK5ttWoUIuLJcOAFDnNXRIvjTlkaUhwKSR3zRNUDuX3Ie+Xd8BZ9RPVkc+1VjrjVkpiEvyJJbIBz6iKnNQX4Aq9ft81SevL+XmnUIcweferGBfQr6vKlH5WDc3dG/QJa/KukhWF9gskf51NRvEHP8ttuRFCyB0qWO5qoZilrdUpSskqyabZwQfvVlYk+zBlq2pUX4zvNBnI6Vxl898mmk/XNpuCT1trSPnvVNxjlQVkjj2NScZs5GFq/U0SxJcnHqG0FtyftsgFcV05V36uKhCk56cZFfW0kJ5Oc0qEAjOaLcipKbl2IoYQD2p5FijryU1qy2Ovmn7LY6hiu70JycqkOIqvIWk4AOe1Ece8LbaSPMPH3PFCTjikODnODS3nrKRg4rpWl7S9rFepN/22fbHWt6zyU4VnJ8tz7/nSGn+uay824VKwPc5pp4eSbhH1TZ1nqVKhgoSeclPYipDTDLkZ+VGcThSMg/oaGftD0/ZFSkJbeW2SDg4wf8qr3X+1dt1QldwhobjzAgYSlICVEfNHt2UROdx80kiUBgdIJx70nFlcXTHZ42ce6g09Nss12FLjqbcbJBB9/uPtUAtBCjgHFdca00PbdXxViQG2pKQS2908g/B+RVD6j2o1ZbH1KRbPOZzkLZ5GMfFamPLujyY+bHtfBXpGBkisR1FXoJx9qeTLfNgqUiZFdaKTj1IIpsjAPUM0xOxO1i4JxzWVqlfVnjtW1dOHo3pwnoRyasfTLLKC1HQ0hLRGOgJATg9+O3Oaysrxj6PquPotWP8A9Ha/uitF/iV+VZWUMCzEjLj/AEX60EXf8R/WsrKNdnX0At+/Ar9ari8fid/M1lZVnH2VsvQNKJ6jye4/zq4bapR0pHyo9j71lZRy7EgBqQnpXye1UlqtR89fJ96ysqxh95U1P1AOV+NX502Pt+dZWVePOy/uD2N7VLxe4rKyuPoYyST+EUqn8IrKykCxVn8Yp+z+IVlZUQuXY3kfj/WlE/hFZWU9FafRdXhc51XPz/8AtzRE2ANRXUAY9av86ysoZ+0PT+4ELv8A9Oc/M0xR/S/rWVlVUWc3QsCfNQM8HORWjyUltZKRnrHtWVlXsfRk5uwWv0KHITMMiIy7hCj62wrn55rl68pSi6SUoSEgOHAAwBWVlW8ftEDRr3/OlKyspgpn/9k=" />
</p>

How do I know it is 3.13? There is an open-source tool called [Decompyle++](https://github.com/zrax/pycdc) that can disassemble and decompile different version of Python uniformingly. Using its disassembler `pycdas`, we can meaningfully speculate the file.

```bash
pycdas --pycode-extra funny.pyc
```

```yaml
funny.pyc (Python 3.13)
[Code]
    File Name: chal.py
    Object Name: <module>
    Qualified Name: <module>
    Arg Count: 0
    Pos Only Arg Count: 0
    KW Only Arg Count: 0
    Stack Size: 7
    Flags: 0x00000000
...
```

Unfortunately, we do not have the same success when using `pycdc`, the decompiler.

```py
# Source Generated with Decompyle++
# File: funny.pyc (Python 3.13)

Unsupported opcode: COPY (218)
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
# WARNING: Decompyle incomplete
```

Here comes the most "respectful" thing about open-source. 80% of the comment on the repo are: _please support xxx feature_ without a corresponding pull request. Cool! I found a [blog post by idafchev](https://idafchev.github.io/blog/Decompile_python/) about hacking the tool to do "fake" support on the opcode, which is basically adding cases in `BuildFromCode` function in ASTree.cpp file. And it works just enough for us to see what's going on:

```py
# Source Generated with Decompyle++
# File: funny.pyc (Python 3.13)

from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
a = 0
b = 0
c = 0
d = 0
e = 0
f = 0
g = 0
h = 0
i = 0
j = 0
k = 0
l = 0
m = 0
n = 0
o = 0
p = 0
q = 0
r = 0
s = 0
t = 0
u = 0
v = 0
w = 0
x = 0
y = 0
z = 0
key = None('Key > ').encode()
iv = None('IV  > ').encode()
if None(key) == 32 and None(iv) == 16:
    decrypted = None(AES.new(key, AES.MODE_CBC, iv).decrypt(...), AES.block_size)
    if b'INTIGRITI' in decrypted:
        None('gg')
    None(decrypted.decode())
del decrypted
del key
del iv
i_wonder_what_this_decrypts_to = ... # Some super long cipher text
(AES.new(..., AES.MODE_CBC, ...).decrypt(i_wonder_what_this_decrypts_to), AES.block_size)
# 1627 lines of just variables?
return None
if ImportError:
    None('run `pip install pycryptodome` to be able to play this challenge')
    None()
```

The decompiler could not put the correct function name, but no worries, as we can fix this by hand. If we look back at the disassembler, we got a list of names used in the program.

`Crypto.Cipher`, `AES`, `Crypto.Util.Padding`, `pad`, `unpad`, `ImportError`, `print`, `exit`, `a`, `b`, `c`, `d`, `e`, `f`, `g`, `h`, `i`, `j`, `k`, `l`, `m`, `n`, `o`, `p`, `q`, `r`, `s`, `t`, `u`, `v`, `w`, `x`, `y`, `z`, `input`, `encode`, `key`, `iv`, `len`, `new`, `MODE_CBC`, `decrypt`, `block_size`, `decrypted`, `decode`, `i_wonder_what_this_decrypts_to`

So I started matching them through guessing:

|Code                                                                   |Guess              |
|:----------------------------------------------------------------------|:-----------------:|
| `None('Key > ').encode()` <br> `None('IV  > ').encode()`              | `input`           |
| `None(key) == 32` <br> `None(iv) == 16`                               | `len`             |
| `None(AES.new(key, AES.MODE_CBC, iv).decrypt(...), AES.block_size)`   | `unpad`           |
| `None('gg')` <br> `None(decrypted.decode())` <br> `None('run ...')`   | `print`           |
| `None()`                                                              | `exit`            |

There is another `AES.new` at the bottom with improper enclosure, so I assumed it does the same thing as above and wrapped it in an `unpad` call.
We then get a better view at the source code:

```py
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
a = 0
b = 0
c = 0
d = 0
e = 0
f = 0
g = 0
h = 0
i = 0
j = 0
k = 0
l = 0
m = 0
n = 0
o = 0
p = 0
q = 0
r = 0
s = 0
t = 0
u = 0
v = 0
w = 0
x = 0
y = 0
z = 0
key = input('Key > ').encode()
iv = input('IV  > ').encode()
if len(key) == 32 and len(iv) == 16:
    decrypted = unpad(AES.new(key, AES.MODE_CBC, iv).decrypt(...), AES.block_size)
    if b'INTIGRITI' in decrypted:
        print('gg')
    print(decrypted.decode())
del decrypted
del key
del iv
i_wonder_what_this_decrypts_to = ... # Some super long cipher text
unpad(AES.new(..., AES.MODE_CBC, ...).decrypt(i_wonder_what_this_decrypts_to), AES.block_size)
```

### A potential shortcut?

A summary of the program is as follow:
1.  User inputs in key and IV ([Wikipedia: Block cipher mode of operation](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Initialization_vector_(IV)))
2.  Check if key length is 32, IV length is 16.
    -   If true, use them to decrypt pre-determined string. Print _gg_ if the decrypted text contains _INTIGRITI_. Then print the decrypted text.
3.  Using a different key and IV, decrypt `i_wonder_what_this_decrypts_to`.

The check in step 2 can easily be bypassed if we patch the string it decrypts to match with something we input. However, step 3 is more tempting and does not require step 2 to be success. So I installed Python 3.13, ran `python -i funny.pyc` (i for interactive), and paste the last line to get decrypted text of `i_wonder_what_this_decrypts_to`.

Here comes the flag. And we have successfully solved the challenge. Thank you for reading the writeup. Like and...

But wait.

```
If you've uncovered this message, you're probably on the right track.

The goal for this challenge is to figure out the correct key and IV to decrypt the flag. You must have noticed all the names being pushed and popped and wondered what they're for...
I promise it's not too scary. The key to decrypt this flag is written in ASCII art using these names, so you must reconstruct the source for everything under this
message decryption down to the exact spacing. The IV is the characters at positions
[(476, 6), (468, 5), (282, 6), (506, 6), (420, 3), (492, 0), (192, 6), (56, 6), (144, 3), (324, 0), (360, 1), (352, 6), (30, 1), (260, 0), (298, 1), (480, 3)]
in that exact order joined together where (0, 0) is the *TOP LEFT* of where the ASCII art begins.

For example, consider the following *FAKE* ASCII art where the IV positions are [(0, 0), (5, 1), (2, 6), (10, 6)]

axxxx   x       xxxxxxxxx
x    b  x           x
x    x  x           x
xxxxx   x           x
x    x  x           x
x    x  x           x
xxcxx   xxdxxx      x

In this example, the key would be "BLT" and the IV would be "abcd" since the character at (0, 0) is 'a', (5, 1) is 'b', (2, 6) is 'c', and (10, 6) is 'd'.

To clarify further, all semicolons are placed "normally" meaning that they're placed directly after a statement with no spacing (ex: a;b;   c;  d;), the final IV will only contain characters a-z, and I highly recommend
using a text editor that uses an equal width for all characters to avoid misalignment.

Good luck!
```

A message from the author! The real flag is in fact in the check earlier and can only be found with the correct key and IV. To do so, we need to guess the ASCII art positioning after the decryption.

### Recovering ASCII art from the line table

If we take a look at the decompiler output from earlier, we can see a long list of variables with no exact uses. We also got the hint that ASCII art is written in the form of `a;b;   c;  d;`. So how do all of these work? One misconception must be cleared first: `;` (semicolon) is meaningful in Python. Functionally, it is very similar to `\n` (new line), but more restricted. You can write `a = 3; b = 4` and it would run as you would expect in other languages. The annoying thing is that you cannot end a line with `;`. So what the bytecode actually tells us is the list of variables, in the order of left-to-right, low-to-high. And to get the coordinates, we need to look at line table.

_Line table_ might be a horrible name, because it is very difficult to search its documentation. Most links point to [PEP 626 – Precise line numbers for debugging and other tools](https://peps.python.org/pep-0626/), which is not helpful for what we are doing. As I was struggling to figure out, I found this StackOverflow [answer](https://stackoverflow.com/a/59431935). In short, for Python 3.13, we can parse pyc using the following code:

```py
import dis
import marshal

with open('funny.pyc', 'rb') as f:
    f.seek(16)
    dis.dis(marshal.load(f))
```

But `dis.dis` would just print out in format similar to what we saw from `pycdas` earlier. Not helpful. However, if we look at its [documentation](https://docs.python.org/3/library/dis.html#dis.dis):

> `dis.dis(x=None, *, file=None, depth=None, show_caches=False, adaptive=False)`
> 
> Disassemble the x object. x can denote either a module, a class, a method, a function, a generator, an asynchronous generator, a coroutine, a code object, a string of source code or a byte sequence of raw bytecode...

So in our case, what is the type of x? I thought it is _a byte sequence of raw bytecode_ so we are limited by the library, but I was wrong. It is in fact _a code object_ as a result of `marshal.load`, which leads us to another awesome [function](https://docs.python.org/3/library/dis.html#dis.get_instructions):

> `dis.get_instructions(x, *, first_line=None, show_caches=False, adaptive=False)`
> 
> Return an iterator over the instructions in the supplied function, method, source code string or code object.
> 
> The iterator generates a series of Instruction named tuples giving the details of each operation in the supplied code.
> 
> ...

This means we can iterate and get every bytecode instruction!

```py
import dis
import marshal

with open('funny.pyc', 'rb') as f:
    f.seek(16)
    a = (marshal.load(f))
print([i for i in dis.get_instructions(a)])

# [Instruction(opname='RESUME', opcode=149, arg=0, argval=0, argrepr='', offset=0, start_offset=0, starts_line=True, line_number=0, label=None, positions=Positions(lineno=0, end_lineno=1, col_offset=0, end_col_offset=0), cache_info=None), ...]
```

Pretty cool, right? It is even better when you realized that among them, there are Position informations!

```py
for i in dis.get_instructions(a):
    pos = i.positions
    # Print name and relevant location in the source code
    print(i.opname, (pos.lineno, pos.col_offset))

"""
RESUME (0, 0)
NOP (1, 0)
LOAD_CONST (2, 4)
LOAD_CONST (2, 4)
...
"""
```

Woah! Did Python leak or exfil-ed the source code? No. Remember that sometimes Python asserts and informs you the line and visualize it using arrows. That is exactly what the position is used for.

```py
File "test.py", line 29, in <module>
    key = input('Key > ').encode()
          ~~~~~^^^^^^^^^^
KeyboardInterrupt
```

Even without the source code, Python can still show you the line to look at.

```py
  File "chal.py", line 9, in <module>
KeyboardInterrupt
```

With that, we can now reconstruct the ASCII art. Since it is located after the decryption part and before the `ImportError`, I limited the offset to be within 600 and 2778. I observed that each of the symbol (`a`, `b`, ...) correlates with a `LOAD_NAME` opcode, and each of the `LOAD_NAME` instruction contains positions information. To simplify the matter, I created a virtual text file `vt` that allows writing to any specific line and column.

```py
for i in dis.get_instructions(a):
    # We only want to look at the list of variables after AES decryption
    if 2778 > i.offset > 600 and i.opname == "LOAD_NAME":
        pos = i.positions
        # Write out the symbol at the exact location.
        # This assumes the symbol has length of 1
        vt.write((pos.lineno, pos.col_offset), i.argrepr)
```

Printing out the reconstructed text, we can find the key we were looking for!

```
b;r;y;b;e;f;        e;j;n;k;i;      d;o;s;b;o;g;u;  
x;          x;    x;          b;    h;              
p;          t;    j;        i;l;    e;              
k;s;d;p;s;b;      j;    m;    g;    e;k;y;y;r;d;    
r;                d;g;        v;                i;  
p;                e;          m;    z;          m;  
o;                  f;y;n;g;r;        b;k;a;l;d;    ...
```

Now, to get the IV, the message from earlier was again helpful to give us a python array to use. We just need to remap into our text file and wallah, we got it!

```py
# From the earlier message
iv_mapper = [(476, 6), (468, 5), (282, 6), (506, 6), (420, 3), (492, 0), (192, 6), (56, 6), (144, 3), (324, 0), (360, 1), (352, 6), (30, 1), (260, 0), (298, 1), (480, 3)]
actual_mapper = [(y + 24, x) for (x, y) in iv_mapper]
actual = ''.join([vt.get(c) for c in actual_mapper])
print(actual)
```

The program should accept both the key and IV we found, and here is the flag we are looking for.

```
gg
INTIGRITI{y0u_7ruly_4r3_7h3_pyc_p4r51n6_m4573r}
```

Interestingly, when I compared with the official writeup after the CTF, the author parses from `code.co_positions` manually rather than using `dis.get_instructions(code)`. I guess the Python documentation is not informational on Python best practices. I'm curious how the ASCII art was created though.

If you want to see the code, scroll down to the [bottom](#final-words), because the next part is...

### Bonus: Whole source code reconstruction

Please hear me out. If I can produce a pyc with the same byte code and line table, then I might have the same source code (minus stuff that are not explicitly or implicitly recorded inside pyc, such as comments). I did this before recovering from the line table, because I thought I need to recover the whole source code file!

Using the source code from earlier (under file name test.py), I ran `python -m compileall test.py` to force it to produce `pyc` file.

At first, I tried to match the bytecode. I used `pycdas` to produce bytecode listing of both files (test.cpython-313.pyc and funny.pyc), and then compared them side-by-side (`vimdiff` or `vscode`, your choice). Let's look at the differences!

| funny.pyc                                 | test.cpython-313.pyc                    |
|-------------------------------------------|-----------------------------------------|
| `LOAD_CONST                      0: 0`    | `LOAD_CONST                      0: 0`  |
| `COPY                            1   `    |                                         |
| `STORE_NAME                      8: a`    | `STORE_NAME                      5: a`  |
| `COPY                            1   `    | `LOAD_CONST                      0: 0`  |
| `STORE_NAME                      9: b`    | `STORE_NAME                      6: b`  |
| `COPY                            1   `    | `LOAD_CONST                      0: 0`  |
| `STORE_NAME                      10: c`   | `STORE_NAME                      7: c`  |
| `COPY                            1   `    | `LOAD_CONST                      0: 0`  |

First, we see `COPY` opcode again. Per documenation, `COPY(i)` would copy the i-th of the stack and push it on top of the stack. How can we produce this opcode? Thankfully, from the Decompyle++ repo, we got a [minimal reproducible example](https://github.com/zrax/pycdc/issues/452#issuecomment-1999660941) for this:

```py
def COPY():
    a = 10
    b = 10
    c = 10
    return a == b == c
```

The expression after return is what produces the `COPY` opcode. For our case, we simply chain the assignment into one line. Here, I also remove space to align with the line table.

```py
a=b=c=d=e=f=g=h=i=j=k=l=m=n=o=p=q=r=s=t=u=v=w=x=y=z=0
```

Next, we see another extra section of bytecode from funny.pyc:

```
2776    PUSH_EXC_INFO                   
2778    LOAD_NAME                       5: ImportError
2780    CHECK_EXC_MATCH                 
2782    POP_JUMP_IF_FALSE               19 (to 2822)
2786    POP_TOP                         
2788    LOAD_NAME                       6: print
2790    PUSH_NULL                       
2792    LOAD_CONST                      3: 'run `pip install pycryptodome` to be able to play this challenge'
2794    CALL                            1
2802    POP_TOP                         
2804    LOAD_NAME                       7: exit
2806    PUSH_NULL                       
2808    CALL                            0
2816    POP_TOP                         
2818    POP_EXCEPT                      
2820    JUMP_BACKWARD_NO_INTERRUPT      1396 (to 32)
```

There are several interesting opcodes here: `PUSH_EXC_INFO`, `CHECK_EXC_MATCH`, `POP_EXCEPT`, `JUMP_BACKWARD_NO_INTERRUPT`. All of them suggest that there might be exception handling.
From earlier link, we can find another helpful reproducible example:

```py
def PUSH_EXC_INFO():
    try:
        raise Exception("This is an exception")
    except Exception as e:
        raise e
```

With a little bit of imagination, I successfully reconstructed them as the following code:

```py
try:
    ...
except ImportError:
    print('run `pip install pycryptodome` to be able to play this challenge')
    exit()
...
```

There are also several parts where the decompiler guesses the indent incorrectly, which can be seen in different jump destination when compare.
For example, in the check, both prints are next to each other, not on different scope:

```diff
     if b'INTIGRITI' in decrypted:
         print('gg')
+        print(decrypted.decode())
-    print(decrypted.decode())
```

Then the `del decrypted` is within the check rather than outside

```diff
 if len(key) == 32 and len(iv) == 16:
     decrypted = ...
     ...
+    del decrypted
-del decrypted
```

Finally, `del key` and `del iv` can be combined into one line to match the line table.

```diff
+del key, iv
-del key
-del iv
```

From this point, only the line table differs. Except for the ASCII art, The rest of the code are guessed through trial-and-error. I used [biodiff](https://github.com/8051Enthusiast/biodiff) to compare pyc side-by-side. After 2 hours, I was able to reconstruct the whole Python source code with no differences in both bytecode and line table! The only difference is the timestamp in the header, which we can ignore. (It is _Thursday, October 31, 2024 3:27:40 PM UTC_, but you can just do time travel in your system)

You can find my final reconstruction here: [dungwinux/1337up-live:funny/chal.py](https://github.com/dungwinux/1337up-live/blob/master/funny/chal.py)

If you have read through this far, congratulation, because you have gone down the rabbit hole of decompiling pyc. I hope you learned something new through the challenge.

### Final words

You can find all of my code here: [dungwinux/1337up-live:funny/](https://github.com/dungwinux/1337up-live/tree/master/funny). I want to give my thank to the author for the wonderful and creative challenge, because I really enjoyed it, and I hope you did as well.
