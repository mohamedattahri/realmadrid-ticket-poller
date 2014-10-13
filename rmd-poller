#!/usr/bin/env python
# coding: utf-8

import re
import time
import urllib2
import hashlib
import smtplib
import sys
from datetime import datetime


game 			= "R-MADRID-CF-FC-BARCELONA-partido_Zonas_55_8_0_17" # 
pollFrequency	= 15 #seconds
smtp_config		= {
		"server"	: "smtp.gmail.com",
		"username"	: "EMAIL",
		"password"	: "PASSWORD",
}
recipients 		= [
	 "someone@address.com",
]

target_url 	= "http://www.entradas.realmadrid.com/entradas/dwr/call/plaincall/FutbolAjax.getPreciosZonas.dwr"

gameIds = [int(s) for s in game.split("_")[2:]]
payload = {
	"callCount"			: "1",
	"page"				: "/entradas/%s" % game,
	"c0-scriptName"		: "FutbolAjax",
	"c0-methodName"		: "getPreciosZonas",
	"httpSessionId"		: "",
	"scriptSessionId"	: "F54D9390EA104A08BCA3B38B75CE598F114",
	"c0-id"				: "0",
	"c0-param0"			: "string:2",
	"c0-param1"			: "string:1",
	"c0-param2"			: "number:%d" % gameIds[0],
	"c0-param3"			: "number:%d" % gameIds[1],
	"c0-param4"			: "number:%d" % gameIds[2],
	"c0-param5"			: "number:%d" % gameIds[3],
	"c0-param6"			: "number:1",
	"batchId"			: "1",
}

regex = re.compile('entradasDisponibles=(?P<number>\d+)(?:.+)literalPrecio="(?P<location>.+)"(?:.+)precio=(?P<price>\d+)(?:.+)propiedades="(?P<category>.+)"')

exclude_vip = False

def send_email(content=None):
	smtp = smtplib.SMTP(smtp_config["server"], 587)
	smtp.ehlo()
	smtp.starttls()
	smtp.login(smtp_config["username"], smtp_config["password"])
	if content:
		smtp.sendmail(smtp_config["username"], recipients, content)
	smtp.close()


def generate_email(availabilities):
	'''email generates a basic report from the list of given availablities, and sends it
	to the recipients configured in the global variable above.'''
	seats  = sum([item["available"] for item in availabilities])

	headers = "\r\n".join(["from: " + smtp_config["username"],
                       "subject: %d seats available" % seats,
                       "to: " + '; '.join(recipients),
                       "mime-version: 1.0",
                       "content-type: text/plain"])
	
	body = ''
	for item in availabilities:
		body += 'Location: %(location)s\nSeats: %(available)d\nPrice: €%(price).2f\nCategory: %(category)s\n\n' % item
	body += 'Buy them here: http://www.entradas.realmadrid.com/entradas/%s&idioma=fr' % game
	
	send_email(headers + "\r\n\r\n" + body)
	print "email has been successfully sent to report the availability of %d seats." % seats


def find_seats(response):
	'''find_seats attempts to find seats info from the raw javascript code returned by the ticketing server.
	Method returns a list of dictionaries highlighting the availability, the price and the location of each ticket
	category available.'''
	result = regex.findall(response)
	if not result:
		return
	availabilities = [{"available": int(hit[0]), "location": hit[1], "price": float(hit[2]), "category": hit[3]} for hit in result]
	if exclude_vip:
		return [item for item in availabilities if item["category"] != "VIP"]
	return availabilities


def are_equal(a, b):
	'''are_equal returns a value indicating whether two sets of availabilities are equal or not.
	Methods uses the hash_availabilities method below.'''
	return hashlib.sha1(str(a)).hexdigest() == hashlib.sha1(str(b)).hexdigest()


def hash_availabilities(availabilities):
	'''hash_availabilities returns a sha1 hash of the list of given availabilities.
	This is useful for a straight forward comparaison of two sets of availabilities.'''
	if not availabilities:
		return None
	return hashlib.sha1(availabilities).hexdigest()


def poll():
	'''poll sends a POST request with parameters found in the Ajax request from the website on Chrome.
	Methods returns the set of availabilities found.'''
	req = urllib2.Request(
		target_url, 
		"\n".join(["%s=%s" % (key, value) for key, value in payload.iteritems()]),
		{
			"Content-Type"		: "text/plain",
			"Host"				: "www.entradas.realmadrid.com",
			"Origin"			: "http://www.entradas.realmadrid.com",
			"Accept-Language"	: "en-US,en;q=0.8,fr;q=0.6",
			"Accept"			: "*/*",
			"Referer"			: "http://www.entradas.realmadrid.com/entradas/%s&tipo=1&identidad=1&" % game,
			"Pragma" 			: "no-cache",
			"Cache-Control"		: "no-cache",
			"User-Agent"		: "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/38.0.2125.101 Safari/537.36",
		}
	)

	try:
		response = urllib2.urlopen(req)
		raw = response.read()
		return find_seats(raw)
	except(urllib2.HTTPError), err:
		print err.code, err.read() 


def main():
	'''main polls the server on a regular basis, and sends an email if the results found between two requests are
	different.'''
	last_poll = None

	# Testing whether the SMTP parameter provided are valid.
	try:
		send_email()
	except(Exception), err:
		sys.exit(err)
	
	print "Polling realmadrid.com for %s" % game
	while True :
		try:
			availabilities = poll()
			if availabilities and not are_equal(availabilities, last_poll):
				generate_email(availabilities)
			last_poll = availabilities		
		except(Exception), err:
			print "Error: %s" % err
		finally:
			time.sleep(pollFrequency)

if __name__ == "__main__":
	main()
