#!/usr/bin/env python

#ID3 tags
import eyed3
#CMD Line args
import sys
#File/dirs
import os
#HTML files
import urllib2
#Verbose output
verbose = True
#String
import string
#Handle sig int
import signal


def die():
	exit()
	raise SystemExit
	sys.exit(-1)

#Handle sig int
def sigint_handler(signal, frame):
        print ""
        print "Ouch!"
        die()
signal.signal(signal.SIGINT, sigint_handler)

v_print_data = ""
def vprint_dump():
	if(verbose):
		print v_print_data

#Verbose print optional
def vprint(s):
	global v_print_data
	#Created a cache of text to print at end
	v_print_data = v_print_data + s + '\n'
	
#Get html content of webaddr w/ err handling
def html_get(web_addr):
	url = web_addr
	data = None
	try:
		usock = urllib2.urlopen(url)
		data = usock.read()
		usock.close()
	except KeyboardInterrupt:
		#Need to quit...
		die()
	except:
		vprint("	Web error.")
	
	return data
	
def html_to_string(html_string):
	#Remove html formatting strings
	html_string = html_string.replace('<br />','')
	html_string = html_string.replace('<br/>','')
	return html_string
	
def get_lyric_text(lyric_page_html):
	#Get rid of evrything before the lyrics
	ly_box = 'lyric-box'
	i = lyric_page_html.find(ly_box)
	lyric_page_html = lyric_page_html[i:]
	#Then the lyric start at the first textblock
	textblock = 'id="textblock"'
	i = lyric_page_html.find(textblock)
	lyric_page_html = lyric_page_html[i:]
	#Then first closing bracket is lyric start
	close_bracket = '>'
	i = lyric_page_html.find(close_bracket)
	lyric_page_html = lyric_page_html[i+len(close_bracket):]
	#The end of lyrics should be the div tag
	div = '</div>'
	i = lyric_page_html.find(div)
	lyric_page_html = lyric_page_html[:i]
	#Finally replace some items
	lyric = html_to_string(lyric_page_html)
	lyric = lyric.strip()
	return lyric


#Get html of song search results
def get_song_results_html(title):
	#Url for search looks like
	#http://www.songmeanings.net/query/?query=float%20on&type=songtitles
	search_url = "http://www.songmeanings.net/query/?query=" + title.replace(" ","%20") + "&type=songtitles"
	return html_get(search_url)
	
def get_artist_results_html(artist):
	#Url for search looks like
	#http://www.songmeanings.net/query/?query=Modest&type=artists
	search_url = "http://www.songmeanings.net/query/?query=" + artist.replace(" ","%20") + "&type=artists"
	return html_get(search_url)


#Get info from tag
#Pull info from next occurance of name="value"
def get_next_property_info(td_tag,info_name):
	start=info_name + '="'
	end='"'
	istart = td_tag.find(start)
	iend = td_tag.find(end,istart+len(start))
	rv = td_tag[istart+len(start):iend]
	return rv

#Get link from td tag
def get_link_from_td_tag(td_tag):
	return get_next_property_info(td_tag,'href')

#Get info from title=""
def get_title_from_td_tag(td_tag):
	return get_next_property_info(td_tag,'title')

#Return [artist,link]
def process_item_tag_artist(tag_html):
	#should only be two td tags
	#First one should be artist
	#Second one is number of lyrics...who cares!
	td_tag_start = '<td '
	td_tag_end = '</td>'
	x1 = tag_html.find(td_tag_start)
	x2 = tag_html.find(td_tag_end)
	first_td = tag_html[x1+len(td_tag_start):x2] #Artist
	
	artist = get_title_from_td_tag(first_td)
	link = get_link_from_td_tag(first_td)
	
	return [artist,link]


#Process table item from artist page
#Return [title,link]
def process_item_tag_artist_page(tag_html):
	#A few td tags
	#First one should be title and link
	td_tag_start = '<td '
	td_tag_end = '</td>'
	x1 = tag_html.find(td_tag_start)
	x2 = tag_html.find(td_tag_end)
	first_td = tag_html[x1+len(td_tag_start):x2] #Title+link
	
	#SM adds " lyrics" to the title...
	title = get_title_from_td_tag(first_td)
	title = title.replace(" lyrics","")
	link = get_link_from_td_tag(first_td)
	
	return [title,link]

#Return [Artist,Title,link]
def process_item_tag_song(tag_html):
	#should only be two td tags
	#First one should be title
	td_tag_start = '<td '
	td_tag_end = '</td>'
	x1 = tag_html.find(td_tag_start)
	x2 = tag_html.find(td_tag_end)
	y1 = tag_html.rfind(td_tag_start)
	y2 = tag_html.rfind(td_tag_end)
	first_td = tag_html[x1+len(td_tag_start):x2] #Title
	second_td = tag_html[y1+len(td_tag_start):y2] #Artst
	
	artist = get_title_from_td_tag(second_td)
	title = get_title_from_td_tag(first_td)
	link = get_link_from_td_tag(first_td)
	
	return [artist,title,link]
	
#Returns list of [artist,song, song_link] from artist page table html
def process_artist_page_table_html(table_html, artist):
	#Already inside the table tag
	#Get item tags
	item_start = '<tr id="lyric-'
	item_end = '</tr>'
	item_start_index = 0
	item_end_index = 0
	rv = []
	while(item_start_index >=0):
		item_start_index = table_html.find(item_start,item_start_index+1)
		item_end_index = table_html.find(item_end,item_end_index+1)
		item_tag = table_html[item_start_index:item_end_index + len(item_end)]
		title,link = process_item_tag_artist_page(item_tag)
		#Must be non null and non empty
		if( artist != None and title != None and link!=None ):
			if( artist != "" and title != "" and link!="" ):
				rv = rv + [[artist,title,link]]
	
	#Done
	return rv

#Returns list of [artist,link] from artist table html
def process_table_html_artist(table_html):
	#Current index
	#Start at the the tbody tag
	tbody_start = '<tbody>'
	tbody_end = '</tbody>'
	body_html = table_html[table_html.find(tbody_start)+len(tbody_start):table_html.find(tbody_end)]
	#Get item tags
	item_start = '<tr class="item">'
	item_end = '</tr>'
	item_start_index = 0
	item_end_index = 0
	rv = []
	while(item_start_index >=0):
		item_start_index = body_html.find(item_start,item_start_index+1)
		item_end_index = body_html.find(item_end,item_end_index+1)
		item_tag = body_html[item_start_index:item_end_index + len(item_end)]
		artist,link = process_item_tag_artist(item_tag)
		#Must be non null and non empty
		if( artist != None and link!=None ):
			if( artist != "" and link!="" ):
				rv = rv + [[artist,link]]
	
	#Done
	return rv


#Returns list of [artist,song, song_link] from song table html
def process_table_html_song(table_html):
	#Current index
	#Start at the the tbody tag
	tbody_start = '<tbody>'
	tbody_end = '</tbody>'
	body_html = table_html[table_html.find(tbody_start)+len(tbody_start):table_html.find(tbody_end)]
	#Get item tags
	item_start = '<tr class="item">'
	item_end = '</tr>'
	item_start_index = 0
	item_end_index = 0
	rv = []
	while(item_start_index >=0):
		item_start_index = body_html.find(item_start,item_start_index+1)
		item_end_index = body_html.find(item_end,item_end_index+1)
		item_tag = body_html[item_start_index:item_end_index + len(item_end)]
		artist,title,link = process_item_tag_song(item_tag)
		#Must be non null and non empty
		if( artist != None and title != None and link!=None ):
			if( artist != "" and title != "" and link!="" ):
				rv = rv + [[artist,title,link]]
	
	#Done
	return rv


#Returns list of [artist,song, song_link] from song search html
def artists_and_songs_from_html_song(song_results_html):
	#Table starts with
	table_start = '<table summary="table">'
	table_end = '</table>'
	#Get text
	istart = song_results_html.find(table_start)
	iend = song_results_html.find(table_end)
	table_html = song_results_html[istart:iend+len(table_end)]
	#Make sure this is the right table
	caption_html = '<caption>songs table</caption>' 
	if(table_html.find(caption_html) > 0):
		#Good get list [artist,song, song_link]
		return process_table_html_song(table_html)
	else:
		vprint("	Title search does not contain a results table.")
		return None

#Loop through list of artists and select the best one
#[artist, artist link]
def best_artist_result(artist_list, artist):
	for a in artist_list:
		sm_artist = a[0].lower().strip()
		artist = artist.lower().strip()
		if(sm_artist == artist):
			vprint("	Found exact artist match.")
			return a[1]
			
	vprint("	No exact artist match found.")
	return None

#Loop through list of [artist,song, song_link] and pick best match
def best_result(artists_and_songs,artist,album,title):
	for asl in artists_and_songs:
		#Lower case and strip everything
		sm_artist = asl[0].lower().strip()
		sm_title = asl[1].lower().strip()
		artist = artist.lower().strip()
		title= title.lower().strip()
		if((sm_artist==artist) and (sm_title ==title)):
			vprint("	Found exact title match!")
			return asl[2]
	
	vprint("	No exact title match found.")
	return None

def isLyricPage(html):
	#Look for lyric box
	if html.find('lyric-box') >=0:
		return True
	else:
		return False
		

#Returns list of [artist,song, song_link] from artist page
def artists_and_songs_from_html_artist(h, artist):
	#Find the song list
	songlist = 'id="songslist"'
	i = h.find(songlist)
	h = h[i:]
	#Limit to the end of that table
	tend = '</tbody>'
	i = h.find(tend)
	h = h[:i]
	#Now just a bunch of tr tags
	artists_and_songs = process_artist_page_table_html(h, artist)
	return artists_and_songs


#Returns list of artist names from artist search html
#[artist,link]
def artist_list_from_search_results(h):
	#FInd the summary table
	summary = 'summary="table"'
	i = h.find(summary)
	h = h[i:]
	#Process the table
	artists_and_links = process_table_html_artist(h)
	return artists_and_links

def isArtistPage(h):
	#Look for bio tag
	bio = 'id="biography"'
	if h.find(bio) >=0:
		return True
	else:
		return False

#Returns list of [artist,song, song_link] from artist search
def artist_based(artist,album,title):
	#Ret val
	artists_and_songs = None
	
	artist_results_html = get_artist_results_html(artist)
	if(artist_results_html==None):
		vprint("	Could not get artist search results html.")
		return None
	
	#May go right to an artist page
	if(isArtistPage(artist_results_html)):
		#Go right to collecting song list from artist page
		artists_and_songs = artists_and_songs_from_html_artist(artist_results_html,artist)
		return artists_and_songs
	else:
		#Need to select artist from results page [artist,link]
		artist_list = artist_list_from_search_results(artist_results_html)
		if(artist_list == None):
			vprint("	Could not get list of artists from artist search.")
			return None
		#Get link
		link = best_artist_result(artist_list, artist)
		if(link == None):
			vprint("	Could not get link to artist page.")
			return None
		link = "http://www.songmeanings.net" + link
		#Get that artist page html
		artist_page_html = html_get(link)
		if artist_page_html == None:
			vprint("	Could not artist page HTML.")
			return None
			
		#Collect song list from artist page
		artists_and_songs = artists_and_songs_from_html_artist(artist_results_html,artist)
		return artists_and_songs


#Return list of [artist,song, song_link] from song search
#OR the lyrics html directly
def title_based(artist,album,title):
	#Do search by song title
	song_results_html = get_song_results_html(title)
	if(song_results_html==None):
		vprint("	Could not get title search results html.")
		return None
		
	#May return right to lyrics page - greturn it
	if isLyricPage(song_results_html):
		vprint("	Found exact title match!")
		lyric_page_html = song_results_html
		return lyric_page_html
	else:
		#Got search results page?
		#Get list of artists and song title [artist,song, song_link]
		artists_and_songs = artists_and_songs_from_html_song(song_results_html)
		if(artists_and_songs == None):		
			vprint("	No list of artists and songs obtained from title search.")
			return None
		else:
			return artists_and_songs


def get_lyric(artist,album,title):	
	#Ultimately want lyrics html to process
	lyric_page_html = None
	#Collect title based and artist based into one...slow but needed?
	artists_and_songs = [] #List of [artist,title,link]
	
	#Artist search normally works better, do first
	artists_and_songs_artist = artist_based(artist,album,title)
	if artists_and_songs_artist==None:
		vprint("	No results from artist based search.")
	else:
		#Add to list
		vprint("	" + str(len(artists_and_songs_artist)) + " results from artist based search.")
		artists_and_songs = artists_and_songs + artists_and_songs_artist
		#Try to pick best result now
		song_link = best_result(artists_and_songs,artist,album,title)
		if(song_link!=None):
			#Got a good link!
			song_link = "http://www.songmeanings.net" + song_link
			#Get lyrics html
			lyric_page_html = html_get(song_link)
	
	#Get title based results if non-exact match from artist search
	if lyric_page_html == None:
		#Get title based results
		#could yield list of lyrics page html
		artists_and_songs_or_lyrics_html = title_based(artist,album,title)
		if artists_and_songs_or_lyrics_html==None:
			vprint("	No results from title based search.")
		elif(type(artists_and_songs_or_lyrics_html) == type("String!")):
			#Returned lyrics html already
			lyric_page_html = artists_and_songs_or_lyrics_html
		else:
			#Just artists and songs LIST, add to list
			vprint("	" + str(len(artists_and_songs_or_lyrics_html)) + " results from title based search.")
			artists_and_songs = artists_and_songs + artists_and_songs_or_lyrics_html
			

	#Did we find the lyrics page html already?
	if lyric_page_html == None:
		#Not yet, still more to do
		#If no results at all
		if len(artists_and_songs) <= 0:
			#NO results at all
			vprint("	No results found.")
			return None
		
		#Pick best result
		song_link = best_result(artists_and_songs,artist,album,title)
		if(song_link==None):
			vprint("	Unable to select best song link to follow from artist search.")
			return None
			
		song_link = "http://www.songmeanings.net" + song_link
		#Get lyrics html
		lyric_page_html = html_get(song_link)
	
	#Should have lyrics html by now
	#Get just lyric text
	if(lyric_page_html != None):
		lyric = get_lyric_text(lyric_page_html)
		return lyric
	else:
		vprint("	Unable to get lyric HTML.")
		return None

#Custom to unicode function
def to_unicode(s):
	#CONVERT LYRIC TO UNICODE!!!
	u = None
	try:
		u = unicode(s, errors='ignore')
	except Exception, e:
		vprint("	Unicode error: " + str(e))
		return u
		
	return u

#Change and return True if sucessful
def change_lyric(audiofile, new_lyric):
	#CONVERT LYRIC TO UNICODE!!!
	u = to_unicode(new_lyric)
	if(u==None):
		vprint("	Unable to convert lyric to unicode.")
		return False
	
	#Many lyrics entries?
	for lyric in audiofile.tag.lyrics:
		#Only change first one
		if lyric == audiofile.tag.lyrics[0]:
			audiofile.tag.lyrics[0].text = u
		#Blank the others
		else:
			lyric = ""
	return True

def remove_bad_chars(s):
	bad_chars = ['!','@','#','$','%','^','&','*','(',')']
	for c in bad_chars:
		s = s.replace(c,'')
	return s
	
def convert_id3_to_2p3(file_path):
	vprint("	Converting to ID3v2.3.")
	#Documentation is so bad I can't find out how to do this in code...
	#Launch command line
	cmd = 'eyeD3 --encoding=utf8 --to-v2.3 "' + file_path + '" &> /dev/null'
	print cmd
	out = os.popen(cmd)
	out.read()
	out.close()
	
def update_re_try(file_path, audiofile, lyric):
	#Re open
	audiofile = eyed3.load(file_path)
	#Change lyric
	if(not(change_lyric(audiofile, lyric))):
		vprint("	Unable to change lyric.")
		return False

	#Try to save again
	try:
		audiofile.tag.save()
		return True
	except Exception,e:
		s = str(e)
		vprint("	Failed to save ID3 tag: " + s)
		return False
	
def update_file(file_path, audiofile, lyric):
	#Update
	if(not(change_lyric(audiofile, lyric))):
		vprint("	Unable to change lyric.")
		return False
	
	#Save
	try:
		audiofile.tag.save()
		return True
	except Exception,e:
		s = str(e)
		vprint("	Failed to save ID3 tag: " + s)
		if(s.find('Unable to write ID3 v2.2')>=0):
			#Convert to 2.3
			convert_id3_to_2p3(file_path)
			#Try again
			return update_re_try(file_path, audiofile, lyric)
		else:
			return False
	
	return True


def processFile(file_path):
	#Open
	audiofile = eyed3.load(file_path)
	#Is mp3 file?
	if(audiofile == None):
		return False
	#Can we read it?
	if audiofile.tag == None:
		print "Unable to read tag: " + file_path
		return False

	#Get info from file
	artist = audiofile.tag.artist
	album = audiofile.tag.album
	title = audiofile.tag.title
	
	#Check info is present
	if artist==None or album==None or title==None:
		vprint("Not all tag info present: " + file_path)
		return False
		
	#Print file info to screen always
	print artist + " - " + title
	
	#Remove non alpha numeric from all fields
	artist = remove_bad_chars(artist)
	album = remove_bad_chars(album)
	title = remove_bad_chars(title)
	
	#Get lyrics
	lyric = get_lyric(artist,album,title)
	if(lyric == None):
		vprint("	Unable to get lyric.")
		return False
	
	#Update the file
	if(not(update_file(file_path, audiofile, lyric))):
		return False
	
	#Done
	return True

def startingDir(dir_path):
	global v_print_data
	vprint("Starting at directory: " + dir_path)
	#Loop through all 
	rootdir = dir_path
	for root, subFolders, files in os.walk(rootdir):
		vprint("Processing directory: " + root)
		#Process files in this dir
		for f in files:
				filePath = os.path.join(root,f)
				#Need this to continue...catch all
				try:
					if(processFile(filePath) == False):
						#Failed, print debug if needed
						vprint_dump()
	
				except Exception, e:
					print "ERROR: " + str(e)
					
				#Reset buffer after every file
				v_print_data = ""
		
		#Reset buffer after every directory too
		v_print_data = ""


				
		
def processFileDir(fd):
	#Is this a single file or a directory
	if os.path.isdir(fd):
		#Is a directory
		startingDir(fd)
	elif os.path.isfile(fd):
		#Is file
		processFile(fd)
	else:
		vprint("What is '" + fd + "', yo?")


def main():
	#Process the arg
	processFileDir(sys.argv[1])
	
	#Done
	sys.exit(0)

#Script begins here, call main
main()
