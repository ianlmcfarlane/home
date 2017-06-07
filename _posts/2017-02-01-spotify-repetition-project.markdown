---
layout: post
title:  "Spotify Song Repetition Visualization Project"
date:   2017-02-01 00:00:00 -0700
categories: posts
---
![](/assets/spotifyapp/robot_rock.png)
*Daft Punk - Robot Rock [https://ianlmcfarlane.github.io/spotifyApp/#/track/4zu9wo2FXoBSsKjO6tRB3R/analysis](https://ianlmcfarlane.github.io/spotifyApp/#/track/4zu9wo2FXoBSsKjO6tRB3R/analysis)*



This app allows you to search the spotify catalog to visualize the acoustic repetition in songs. It was built with React, mobX, and React Router as well as some other smaller JS libraries. For a full list of dependecies, see [https://github.com/ianlmcfarlane/spotifyAppDev/blob/master/package.json](https://github.com/ianlmcfarlane/spotifyAppDev/blob/master/package.json). I may be adding more functionality in the future...

## Project Impetus

This project started out in the hopes of exploring user listening data since Spotify bungled their 'Year in Review' for 2016. In the past users were provided with listening stats for the previous year and as a heavy Spotify user, I wanted to see my usage. Alas, Spotify does not expose such valuable/interesting information via their API. However, the Spotify API is very easy to use and, as it turns out, provides some extremely cool audio analysis data!

## Data

The endpoint in question is documented [here](https://developer.spotify.com/web-api/get-audio-analysis/). For a given track, it returns a sizeable  JSON object that quantitatively approximates how the song sounds so that Spotify may programmatically create playlists, recommendations, etc. The  attribute I was most interested in was that of 'Segments.' Segments are small (<1s) chunks of the song containing duration, multiple decibel levels, a 1x12 pitches array, and a 1x12 timbre array with a confidence level. This was a great start, but with zero documentation on this object provided by Spotify, it wasnt going to be very useful. After some sleuthing, I discovered that this object was the product of Echo Nest, which was acquired by Spotify in 2014. After this I was able to dig up some old Echo Nest documentation that was very useful ([https://ianlmcfarlane.github.io/assets/spotifyapp/echonest_analyzer_docs.pdf](/assets/spotifyapp/echonest_analyzer_docs.pdf))

## Method

Many of the challenges surrounding this project were related to learning React.js. The central problem still remained: how to efficiently compare each segement to determine if it was repeated. Since each segment is multidimensional and each segment attribute was also multidimensional, a brute force loop through each array of arrays would get operationally intensive very quickly. Instead I had to come up with a different comparison algorithm.

The basic idea was to loop through the array and compare the current element to elements already passed over. Pretty Standard. I only wanted to find the most recently repeated segment, I did not need to find all matches, hence `Array.lastIndexOf(element)`. Below is the basic function for comparison. The array pitch/timbre comparison functions calculate cosine similary `mathjs.dot(a,b)/(mathjs.norm(a)*mathjs.norm(b))` which is esentially the amount of vector **a** that is in **b** when **a** is projected onto **b**.


```javascript
for (let i = 4; i < arr.length-1; i++){ //average 5 segment window
	const window_average = average(i, arr.slice(i-2,i+3));
	lastIndex = arrSeen.map((obj)=>{
		return obj.loudness_max.toFixed(0); //truncate precision of max decibel level
	}).lastIndexOf(window_average.loudness_max.toFixed(0)); //check if already seen this decibel level
	if (lastIndex != -1 //check decibel level match, pitch and timbre array smilarity
		&&isPitchSimilar(arrSeen[lastIndex],window_average)
		&& isTimbreSimilar(arrSeen[lastIndex], window_average))
		{
			arrReturn.push({ //add segment match to matches array
				height: -1*arrSeen[lastIndex].loudness_max, 
				section: arrSeen[lastIndex].section_index % 2 ? 'Red' : 'Black',
				startindex: lastIndex,
				outerradius: (i-lastIndex)/2,
				opacity: (window_average.confidence+arrSeen[lastIndex].confidence)/2,
				weight:  .3,
				rotation: "-90",
				startindex: lastIndex
			});
		}
	arrSeen.push(window_average); //add current window to seen array
}
```

With the ouput array of index pairs, it was a simple matter of mapping the segment indexes to pixels scaled to the length of the song (the segements are not actually equal, but for simplicities sake, I assumed they were). With a little D3.js this was pretty simple. I also plotted the max decible 
level of each segment to give some context to the repetition/arcs. I think that one of the best songs that I have come across is Daft Punk - Robot Rock (linked at the top of the post).

## Shortcomings

The first problem with attempting to find repetition is that the song itself is a superpostition of many different repeating intruments. In any given segment, the different instruments may collide in different ways, leading to decible level change or maybe slightly different timbre. Unless you have MIDI files for a song, this is really just the nature of the post production game.

The writeup is still in progress. There is more to come, but still take a look at the app. [Check it out!](https://ianlmcfarlane.github.io/spotifyApp/) FYI you need a Spotify account.