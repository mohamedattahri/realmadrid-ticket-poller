Real Madrid Ticket Poller
========================

Tired of hitting refresh hoping that someone will cancel his purchase so that you can buy his ticket?

This script regularly checks the website for available seats, and sends an email if any are found.


## How to

- Set the game by extracting the game string from the store URL:

	For example:

		http://www.entradas.realmadrid.com/entradas/R-MADRID-CF-RAYO-VALLECANO-partido_Zonas_55_8_0_18&idioma=fr

	The game string here is:

		R-MADRID-CF-RAYO-VALLECANO-partido_Zonas_55_8_0_18

- Set up the SMTP parameters (server, username and password)

- Add email addresses in the recipients list

- Adjust the polling frequency

- Run the script:

		python rmd-poll.py



