# Better with Messages
Email does not feel repsonsive. Designed as an async communication tool. We brought emssages into collaboration to supplement email.

Use existing conversations.  No need to ask for email addresses.  Effective communication is the key to successful collaboration.  Turn into FT calls.  Using screen sharing can make collaboration omre productive.
# New collaboration flow
Share recipes with each other.  Pages document and I'd like to share with friends.  Start by tapping on the sahre button in the toolbar.  

Share in two different ways.
1.  Start a new collaboration
2. send copy

collaborate => messages, mail, "other communication apps".

Selecta  group to start a collaboration.  Send button keeps collaboration immediately.  Cupcake designer's photo appears.  Conversation is now linked to this.

Tapping on group brings up the "collaboration popover".  EAsy access to communication tools.  Pages displays who is currently in the document.

Messages conversation presents over shared document.  If I need to talk to people over facetime, I can simply tap on the overview/video popover.

Tight integration with messages unlocks super powers.  Messages can display notifications to apps.  This mesasge banner allows me to directly open the document.  Now have a strong connection to make collaboration workflows more streamlined.

People can easily drop invitations in conversations, and the popover allows people to jump back to emssages conversations or start ft calls.

Lets people stay up to date.

# Design considerations
* adopt system share sheet
	* people are familiar
	* conversation suggestsions help people find destinations
	* easy way to begin a collaboration
	* more consistent, more familiar
* place share button in your app.  Where people can access.
	* toolbars
* Popup button allows people to choose how to share files.
	* everyone can make changes => entrypoint to collaboration permissions.
		* be as concise as possible
		* when customizing the screen, keep the structure of the choices simple so people can skim options
		* macOS was redesigned to reflect iOS.
* Optimize for messages.
	* customize collaboration settings from input field.
	* don't go back to share sheet to change options.
	* drag-and-drop to add messages is another way to start a project.
	* Help people set permissions from messages.
* Place collaboration button
	* collaboration button will appear in your app.
	* Most important UI elements of the entire collaboration experience
	* find a place where it can stand out.
	* place right next to share button.
	* appearance is different depending on how people started collaboration.
		* recipient's picture?
		* coversation photo?
		* system-provided symbol
* Customize the popover
	* top section: who you are collaborating with.  System will find pictures, etc.  Buttons for communication.  If there is no message conversation, the message button will let you connect one.
	* Middle section: fully customizable.  think about what information or functiosn will be essential for people using your app.
		* we show participant list in pages
		* notes shows latest actions to not overload
		* make sure what you're adding is visually consistent
		* ok to leave this empty like reminders and files
	* Bottom: manage shared file
		* Customize the button label if needed
		* Add/remove participants
		* change settings.
		* For CK, provided by system
		* otherwise, have your own screen.
		* same design on macOS.
* Curate messages banners
	* Allow people to find new updates, without having to open the document.
	* Templates to choose from
		* edits made
		* comments added
		* you were mentioend
		* files modified
		* tell messages what you want to display.
		* what about multiple edits and documents?  Messages will consolidate banners.
		* separate panel
# Wrap up
Adopt system share sheet
Customize collaboration popover
Leverage messages integration

