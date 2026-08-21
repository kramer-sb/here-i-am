# Sisterhood of the Traveling Packets

**Date:** 2026-08-17
**Category:** CTF
**Target:** Flare / SANS / WiCyS "Sisterhood of the Traveling Packets" 
**Tools:** `Tor` `CyberChef` 

## Summary

Flare CTF in partnership with SANS and WiCyS. Beginner-friendly, with nothing but nerve & the Tor browser needed. 

## Steps

1. **Recon**:  Look at the three *visible* pages: ``index.php``, ``crew.php``, and ``about.php``. 
   - ``index.php`` - right away I looked at the page source and found the base64 string that was a faux flag: ``not_the_flag_keep_looking``
   - crew included vex: operator / panel dev, crypt / payload engineer, mora / negotiations, and skid/initial access
   - ``about.php`` - nothing worth noting
2. **Exploitation**: use robot.txt and see that two additional pages may be useful: ``/api.php`` and ``/admin.php``. 
   - Checked out admin.php first and foolishly tried the crew names with some of the usual suspect passwords. No dice!
   - I've never had to break into an api.php page that I can recall, so I tried it, not knowing what to expect. I discovered that the "action" parameter was needed, as well as one of the valid actions: upload, status, messages, decrypt, wallets, payload.
   - ``/api.php?action=messages`` was my first stop because I was positive I would find something fun. I looked through all five (and tried a whole bunch more, just in case), beginning with 0 because that was the example given: ``api.php?action=messages&conversation_id=0``. Up through to ``id=4`` I was able to view messages. *Read them and tbh, I didn't notice at first but Mora's password hash was in ``id=2``* 
   - ``/api.php?action=status`` - nothing pertinent to the challenge, but revealed version, status, nodes, active campaigns, queue, and storage details. 
   - ``/api.php?action=wallets`` - Bitcoin wallets
   - ``/api.php?action=payloads`` - nothing pertinent to the challenge
   - ``/api.php?action=decrypt`` - went down a rabbit hole trying to determine what could be used for the following parameters: ``"error": "missing required parameter: victim_id", "detail": "provide the victim_id from the negotiation channel",  "example": "NXV-2026-041"``
3. **Post-exploitation / Privesc**: Ultimately, the key to the challeng was contained in ``conversation_id=2`` with Mora which contained ``UGFudGFsMG4zc19SdWwzeiE=``. The message explicitly said it was Mora's password and that it had been encoded. The simplest first step is Base64 decoding, ``Pantal0n3s_Rul3z!``
4. **Root/Flag/Conclusion**: The flag was hidden in the admin panel: ``flare{pantal0n3s_g0t_pantsed_2026}
``
## Lessons Learned

- Immediately try the endpoint ``robots.txt``. It had been some time since I participated in a CTF, and I forgot this very basic, yet critical first step. 
- I learned how to format ``api.php``! That's useful information. 
- **Main lesson: follow the breadcrumbs literally before assuming the challenge requires sophisticated cryptography or exploitation.**  

## Notes

I have not properly documented a CTF before. I was so overwhelmed by my CTFs with WiCyS that I was just moving quickly, breaking things, and not keeping good save-with-someone-else notes. Then, for the [Dod Cyber Sentinel Challenge](https://app.notion.com/p/DoD-Cyber-Sentinel-Challenge-212cae110e9e80baa86bdff5d62943c8), I kept notes in my notion space. I love docs for things I am giving to other folks (reports, How-To's, etc.), and I need to be as good about documentation for my own projects in a more methodical way. Of all the things that happened with this CTF, this is my biggest personal takeaway.

## References

- [CyberChef](https://gchq.github.io/CyberChef/)

