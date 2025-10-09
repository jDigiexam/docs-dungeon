# press-them
8-Bit Office Mystery - Game Project
1. Project Overview
This document outlines the architecture and setup for a browser-based, real-time multiplayer mystery game. Players form teams of two to investigate a crime scene, find clues to locate a hidden weapon, and defeat a randomly assigned monster (who is also a player). The game features 8-bit graphics, a mobile-first control scheme, and includes real-time chat and WebRTC-based video/audio communication.

This project uses Vanilla JavaScript for the frontend and Supabase for the backend.

2. Architecture
Frontend (Single index.html file)
Rendering: An HTML <canvas> element is used for rendering the game world, players, and clues.

UI: A simple UI overlay is built on top of the canvas for the lobby, chat, video feeds, and game status. Styling is handled by Tailwind CSS.

Controls:

Mobile: A virtual joystick is implemented using touch events for intuitive movement.

Desktop: Keyboard controls (WASD or arrow keys) are included for development and testing.

State Management: A global gameState object in the client holds all necessary information (player positions, clue locations, game status). This state is synchronized with other players via Supabase Realtime.

Backend (Supabase)
Supabase provides all the necessary backend services for this game.

Authentication: While you can start with anonymous users, Supabase Auth can manage player profiles and identities.

Database (Postgres): Used to store persistent data that doesn't change during a game.

profiles: Stores player display names, character appearance choices, etc.

game_lobbies: Stores information about available game rooms before a game starts.

Realtime: This is the core of the multiplayer experience. We will use channels to broadcast and receive low-latency messages.

game-state channel: Used to broadcast player movements, actions (collecting clues, attacking), and status changes (alive/dead). Every player sends their updated state several times a second, and listens for updates from all other players.

webrtc-signaling channel: WebRTC needs a way for peers to exchange connection information (called "signaling"). This channel will be used to broadcast SDP offers/answers and ICE candidates to establish peer-to-peer video/audio connections.

chat channel: A simple channel for broadcasting chat messages to all players in the game.

3. Supabase Setup
Step 1: Create a Supabase Project

Go to supabase.com and create a new project.

Navigate to your project's Settings > API.

Find your Project URL and anon (public) key. You will need these for the index.html file.

Step 2: Create Database Tables
Go to the SQL Editor in your Supabase dashboard and run the following SQL to create the necessary tables.

-- Create a table for basic player profiles
CREATE TABLE public.profiles (
  id uuid NOT NULL,
  username text,
  avatar_url text,
  updated_at timestamptz,
  CONSTRAINT profiles_pkey PRIMARY KEY (id),
  CONSTRAINT profiles_username_key UNIQUE (username)
);

-- Set up Row Level Security for profiles
ALTER TABLE public.profiles ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Public profiles are viewable by everyone." ON public.profiles FOR SELECT USING (true);
CREATE POLICY "Users can insert their own profile." ON public.profiles FOR INSERT WITH CHECK (auth.uid() = id);
CREATE POLICY "Users can update their own profile." ON public.profiles FOR UPDATE USING (auth.uid() = id);

-- Create a table for game lobbies
CREATE TABLE public.game_lobbies (
  id uuid DEFAULT gen_random_uuid() NOT NULL,
  created_by uuid REFERENCES public.profiles(id),
  lobby_code text UNIQUE,
  is_active boolean DEFAULT true,
  created_at timestamptz DEFAULT now(),
  CONSTRAINT game_lobbies_pkey PRIMARY KEY (id)
);

-- Set up Row Level Security for lobbies
ALTER TABLE public.game_lobbies ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Lobbies are viewable by everyone." ON public.game_lobbies FOR SELECT USING (true);
CREATE POLICY "Authenticated users can create lobbies." ON public.game_lobbies FOR INSERT WITH CHECK (auth.role() = 'authenticated');
CREATE POLICY "Lobby creators can update their own lobby." ON public.game_lobbies FOR UPDATE USING (auth.uid() = created_by);

Step 3: Enable Realtime

Go to Database > Replication.

Ensure replication is enabled. You can toggle it on for your tables.

4. Next Steps & Development Roadmap
This project provides the foundation. To build the full game, follow these steps:

Implement Lobby UI: Create the UI for players to enter their name, choose an appearance, and create/join a game lobby using the game_lobbies table.

Core Game Logic:

Randomly assign one player as the monster when the game starts.

Implement the clue collection system.

Code the win/loss conditions (monster kills all players vs. players find weapon and kill monster).

Implement the revive/respawn mechanic.

WebRTC Integration:

Use the webrtc-signaling channel to exchange SDP and ICE candidates.

Write the JavaScript to create RTCPeerConnection objects for each player in a team.

Attach the remote media streams to the video elements in the UI.

Refine Game Map & Assets: Design your 8-bit game map and character sprites. You can draw these directly on the canvas or use a simple sprite sheet.

Deploy: You can easily deploy this single index.html file using GitHub Pages or any static web host.
