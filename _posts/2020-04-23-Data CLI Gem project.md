---
layout: post
title: Data CLI Gem Project Flatiron School (mod1 project)!
---

# CLI Data Gem Portfolio Project

Looking at today's date I realize that it is exactly a month ago that I started my Part-Time Software Engineering track at Flatiron School. Before that date all my programming was trial and error and googling for some hobby projects. But so far enjoying the adventure.

Now in the program for a month we came to the point that it is time for our first portfolio project. Where to start??

Luckily the project requirements were supplied and are:

* Provide a CLI
* Your CLI application must provide access to data from a web page.
* The data provided must go at least one level deep. 
* Use good OO design patterns. You should be creating a collection of objects, not hashes, to store your data.


**First Step**

Where to begin and what to choose as a project. I decided pretty quick to use the hitlist of the Dutch Top40 listed on [http://top40.nl](http://) Guess being dutch pushed me in that direction, it also helped that it contains a listing that could be scraped.


**Second Step**

Planning the gem was the next step, what Classes to use and how they would interact. I used [http://draw.io](http://) to create an overview how I thougt my program would look like

[http://github.com/raspimeteo/dutch_top40/blob/master/dutch_top40_flowchart.jpg](http://)


**Third Step**

In the project's page posted YouTube clip from Avi he pointed out pretty well how to get started to build a gem and create a skeleton for your program. Doing this through Visual Studio Code was pretty simple with the given instructions. 
After running the 'bundle gem dutch_top40' command bundler creates a complete directory tree for the program. Left only to put the necessary requirements in place like 'open-uri' and nokogiri' that are needed for the scraping. For debugging I also added pry to be able to see how things are working within the program.

Time to create a repository on github and push the project to github. Unfortunately just this week github was dealing with some outages that made github acces unreliable. But I managed to get things done.


**Fourth Step**

Time to put things in their place. As from the flowchart I needed to create three classes CLI, Songs, Scraper with their methods.
I started out with the CLI class and used fake data and some simple puts commands to see if things interconnected were still outputting the expected output.
After the CLI was working and gave me the options I neede it was time to start with the Songs class. Looking at the website of the top40 I saw the info I could obtain from one page. As the Songs class is actually the workhorse of the app I was a bit too focused on obtaining the data and by mistake placed also the scraper in the Songs class.
The website has two options to obtain the list. The mainpage or one level deeper on http://top40.nl/top40. As the second page displayed the info on the site a bit clearer than the main page I tried to scrape the data from that one.
All was working and found the tags to use, only to find out there was another column row that had the same tags and was causing an empty array element. This caused my post-scraping to fail. Needless to say this has cost me some time. After all I went back to the main page, meanwhile also used the repl.it Scraper Check Tool and found a better result on the main page.

After my scheduled meeting with our cohort-lead she also pointed me that I should put the scraper in its own Class.

For a nicer appearance I converted the top40 logo to ASCII to be able to use it as a logo in my CLI class.

```
class DutchTop40::CLI

    def call
        logo
        
        puts "One moment, acquiring data.",""
        list_songs
        # binding.pry
        menu
    end

    def list_songs
        @songs = DutchTop40::Songs.list
        print_songs
        puts
    end

    def print_songs
        puts "Dutch Top40 - week  #{Time.now.strftime("%U")}", ""
        @songs.each.with_index(1) do |song, index| 
            puts "#{index}.  #{song.title}"
        end
    end

    def menu       
            input = nil
            while input != 'exit'
                puts "Which song do you want more info on? Type list to see list again, type exit to quit.",""
                input = gets.strip.downcase
                case input.to_i
                    when 1..@songs.size
                        puts "------------------------------------------------------------------------------------"
                        puts "Current rank #{input}."
                        puts "------------------------------------------------------------------------------------"
                        puts "#{@songs[input.to_i-1].title} - performing artist(s): #{@songs[input.to_i-1].name}"
                        puts "weeks in Top40: #{@songs[input.to_i-1].listed} - last weeks rank: #{@songs[input.to_i-1].last_weeks_rank}"
                        puts "------------------------------------------------------------------------------------"
                    else 
                        puts "Invalid input!" unless input == 'exit' || input == 'list'
                    if input == 'list' 
                        print_songs
                    end
                    if input == 'exit' 
                        goodbye
                    end
                end
            end
    end

    def goodbye
        logo
        puts "See you next time..."
    end

    def logo
        puts "                                                                                                    
        ** logo  removed because the blog markup messed up the way it was displayed **"
    end
end
```



As you can see the CLI is pretty straightforward. The program is started and the bin file calls the call method with DutchTop40::CLI.new.call. In the call method the first action is a call to the logo method to display an ASCII representation of the top40-logo next a call to the list_songs method is made. This list_songs method in its turn calls the list method in the Songs class. After returning from there the returned array of objects is stored in @songs. Next is a call to the print_songs method. In the print_songs method the program iterates over the array and prints out a list of the song titles. Next line of code will call the menu method. The menu is dynamic although my expected output is always 40. But it simplified things not to have to program all 40 options for the song titles. At the end there is a check on invalid input which also gives room for the two options of the list and exit choice.

```
class DutchTop40::Songs

    attr_accessor :title, :name, :listed, :last_weeks_rank

    @@songs = []

    def initialize(title, name, listed, last_weeks_rank)
        @title = title
        @name = name
        @listed = listed
        @last_weeks_rank = last_weeks_rank
        @@songs << self
    end


    def self.list
        DutchTop40::Scraper.scrape_songs
        @@songs
    end

end
```

After removing the scraper from the Songs class it is slimmed down pretty much. Its basic duty now is when the call for the list method comes in to make a call to the Scraper class to do the actual scraping with the scrape_songs method.

The Scraper::scrape_songs method will call the initialize method to create new song objects with the supplied variables. The initialize method also stores the create song objects in @@song. After the scraping is finished the list method will return the @@songs array to the CLI class.
In this class I needed the attribute accessors to create the methods to store the variables.

```
class DutchTop40::Scraper

    attr_accessor :title, :name, :listed, :last_weeks_rank

    def self.scrape_songs
        doc = Nokogiri::HTML(open("http://top40.nl"))
        doc.search('.listScroller .list-right').each do |song|
            # binding.pry
                title = song.css('.songtitle').text.strip
                name = song.css('.artist').text
                details = song.css('.details').text
                listed = details.split(' | ')[1].gsub(/Aantal weken: /,'').strip
                if details.split(' | ')[0].gsub(/Vorige week: #/,'').strip  == '-'
                    last_weeks_rank = 'new entry'
                else
                    last_weeks_rank = details.split(' | ')[0].gsub(/Vorige week: #/,'').strip
                end
                DutchTop40::Songs.new(title, name, listed, last_weeks_rank)
        end
    end
end
```

Scraping as described above was the most challenging part, but with the above listed code I was able to obtain all the info I needed. I could go one level deeper in scraping as the song info also contains a url. That url is leading to a page with some general information about the performing artist. But that would need a second scraping routine to obtain that information.
On the bottom of the scrape_songs method you can see the call to DutchTop40::Songs.new with passing of the title, name, listed and last_weeks_rank. This will results that the Songs class creates an array of objects with all the objects that are created with this Songs.new command.

**Final step**

The final step was beautifying the CLI class so it would give a nice look to the interface and the way the provided info is presented.
If you would like to see how it works you can either download the dutch_top40.gem on [rubygems.org](http://) or download the repo at [https://github.com/raspimeteo/dutch_top40](http://)


