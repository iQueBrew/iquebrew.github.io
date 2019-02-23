# Using the ticket.sys editor and iQueCrypt to add games

### Prerequisites:

1. ticket.sys editor
2. the ticket (.dat), CMD (.cmd) or contentDesc (.cdesc) of the game you want to add
3. the .z64 of the game you want to add
4. HackIt Patcher setup on your console
5. hackit.sys from your console
6. iQueCrypt v1.2.1

### Procedure:

1. Open ticket.sys editor, open hackit.sys in it
2. If you only have a .cmd or .cdesc for the game you want to add, follow the bullet points following this. Otherwise, follow the other ones that are down a bit further.
  * Create a new ticket (Edit → New ticket, shortcut Ctrl+N)
  * In the General tab of the editor, click 'Replace ticket data'
  * Select the .cmd or .cdesc of the game you wish to add and open it

    Now proceed to step 3

  * In the Edit tab, click 'Import ticket.dat' (shortcut Ctrl+I)
  * Navigate to the .dat of the game you wish to add and open it

    Now proceed to step 3

3. Come up with a cool CID and fill in the `CID:` box with it
4. Save the edited hackit.sys (File → Save, shortcut Ctrl+S)
5. Open a command prompt in the same folder as iQueCrypt
6. Run this:
```
iquecrypt.exe encrypt -app <path to the .z64 of the game here> -key 00000000000000000000000000000000 -iv 00000000000000000000000000000000
```
(replace <path to the .z64 of the game here> with the path to the .z64 of your game of choice)
7. Rename `[enc]<game filename here>.z64` to <CID from earlier>.app
8. Copy hackit.sys and <CID>.app to Windows XP and write them to your console with iQueDiagExtend
9. On the console, launch the HackIt Patcher and enjoy your new game!
