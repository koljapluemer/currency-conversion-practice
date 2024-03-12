### Lessons Learned

- [[when having multiple projects with similar names (or multiple versions), make extra clear which is which]] 
	- → see failed *_currency_practice*, had to manually run both to see that this here is the working repo
- interesting decision between an automated currency list or just letting the user put in the transfer rate...
    - with black market rates and all that, there is *a lot* of list here (and annoying to get)
    - then again, putting your own rate is ofc inconvenient...
- [[don't underestimate the complexity of currency]]
    - even with a simple app such as this, you can't just do `toFixed()` if you don't just want hard-coded EGP/EUR