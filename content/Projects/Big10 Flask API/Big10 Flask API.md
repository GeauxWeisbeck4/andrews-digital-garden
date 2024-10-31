---
id: 01JBG4ZY71G20S03BJ6M6XMW99
modified: 2024-10-30T22:29:19-04:00
title: Big10 Flask API
description: An API built with Flask that shares stats, highlights, and more
tags:
  - api
  - flask
  - big10
  - cfb
  - project
---
# Big 10 Flask API

- An API built with Flask
- Has all the Big10 teams with their team name, record, stats, and highlights.

# v1.0.0
- All of the Big10 teams data schema:
	- schoolName: String
		- ex - Nebraska
	- teamName: String
		- ex - Cornhuskers
	- wins: Number
		- ex - 5
	- losses: Number
		- ex - 3
	- nextGame: Object(JSON)
		- ex - 
```json
{
	"game": {
		"opponent": "UCLA",
		"location": "Lincoln, NE",
		"date": "11-02-2024 12:00:00EST",
		"tvChannel": "FOXSN",
		"opponentWins": 3,
		"opponentLosses": 5,
		"spread": +7.5
	}
}
```
- 