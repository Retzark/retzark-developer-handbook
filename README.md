
# Retzark: A Comprehensive Guide for Game Developers

## Table of Contents
1. [Matchmaking and Game Setup](#1-matchmaking-and-game-setup)
2. [Card Selection and Hash Submission](#2-card-selection-and-hash-submission)
3. [Wagering System Implementation](#3-wagering-system-implementation)
4. [Revealing Cards](#4-revealing-cards)
5. [Battle History Simulation](#5-battle-history-simulation)

## 1. Matchmaking and Game Setup

### 1.1 Joining the Waiting Room

#### 1.1.1 Custom JSON on Hive Blockchain

When a player wants to join the waiting room, the app must post a custom JSON operation to the Hive blockchain.

##### Custom JSON Structure:

```json
{
  "required_auths": [],
  "required_posting_auths": ["player_username"],
  "id": "RZ_JOIN_WAITING_ROOM",
  "json": "{\"username\":\"player_username\"}"
}
```

- `required_posting_auths`: Array containing the player's username.
- `id`: Always "RZ_JOIN_WAITING_ROOM" for this operation.
- `json`: A stringified JSON object containing the player's username.

Use the `broadcast.json` method from a Hive blockchain library to post this custom JSON.

#### 1.1.2 POST Request to Retzark Engine

After posting the custom JSON, send a POST request to the Retzark engine to confirm joining the waiting room.

##### Endpoint:
```
POST https://retzark-engine.retzark.com/match/joinWaitingRoom
```

##### Request Body:
```json
{
  "txID": "transaction_id",
  "player": "player_username"
}
```

- `txID`: The transaction ID of the custom JSON operation posted to the Hive blockchain.
- `player`: The username of the player joining the waiting room.

### 1.2 Matchmaking Process

#### 1.2.1 Polling for Match Status

The app should periodically poll the Retzark engine to check if a match has been found.

##### Endpoint:
```
GET https://retzark-engine.retzark.com/player/{username}
```

#### 1.2.2 Loading the Match

Once a match is found, load the match details using the provided `matchId`.

##### Endpoint:
```
GET https://retzark-engine.retzark.com/match/{matchId}
```

### 1.3 Fetching Player Profile Pictures

#### 1.3.1 Retrieve Account Metadata

To get the profile picture URL, first retrieve the account metadata from the Hive blockchain.

##### Endpoint:
```
POST https://api.hive.blog
```

##### Request Body:
```json
{
  "jsonrpc": "2.0",
  "method": "condenser_api.get_accounts",
  "params": [["player_username"]],
  "id": 1
}
```

#### 1.3.2 Extract Profile Picture URL

Parse the `posting_json_metadata` field from the response to extract the profile picture URL.

#### 1.3.3 Download and Display Profile Picture

Use Unity's `UnityWebRequestTexture` to download the profile picture and display it in the UI.

### 1.4 Displaying Initial Game State

#### 1.4.1 Empty Card Slots

When the game starts, all card slots for both players should be empty. Do not display any cards in the slots initially.

#### 1.4.2 UI Updates

- Update the UI to show both players' usernames and profile pictures.
- Ensure all card slots are visibly empty for both players.
- Display any relevant game information such as player energy, base health, etc.

#### 1.4.3 Preparing for Card Selection

While no cards are displayed, the app should prepare for the card selection phase:
- Load the player's deck information.
- Set up UI elements for card selection (e.g., a separate deck view or selection interface).
- Implement logic to handle card selection and placement when the appropriate game phase begins.

## 2. Card Selection and Hash Submission

### 2.1 Card Selection Interface

- Display the player's deck or a subset of cards available for selection.
- Allow the player to select up to two cards for the first round.
- Provide a clear visual indication of selected cards and remaining selection slots.
- Implement a confirmation mechanism to finalize the selection.

### 2.2 Generating Card Hash

Once the player has selected their cards, generate a hash of the selection:

1. Create a string representation of the selected cards:
   - Use card IDs or another unique identifier for each card.
   - Example: If the player selected cards with IDs 12 and 45, the string might be "12,45".

2. Generate a SHA-256 hash of this string:
   - Use a cryptographic library to create the hash.
   - The resulting hash should be a 64-character hexadecimal string.

### 2.3 Posting Card Hash to Hive Blockchain

Post the generated hash to the Hive blockchain as a custom JSON operation:

#### Custom JSON Structure:

```json
{
  "required_auths": [],
  "required_posting_auths": ["player_username"],
  "id": "RZ_CARD_SELECTION",
  "json": "{\"hash\":\"generated_card_hash\"}"
}
```

- `required_posting_auths`: Array containing the player's username.
- `id`: Always "RZ_CARD_SELECTION" for this operation.
- `json`: A stringified JSON object containing the generated card hash.

Use the `broadcast.json` method from a Hive blockchain library to post this custom JSON.

### 2.4 Submitting Card Hash to Retzark Engine

After posting the hash to the Hive blockchain, submit it to the Retzark engine:

#### Endpoint:
```
POST https://retzark-engine.retzark.com/match/submitCardsHash
```

#### Request Body:
```json
{
  "txID": "transaction_id",
  "player": "player_username"
}
```

- `txID`: The transaction ID of the custom JSON operation posted to the Hive blockchain in step 2.3.
- `player`: The username of the player submitting the hash.

### 2.5 Waiting for Opponent

After submitting the hash:

- Display a "Waiting for opponent" message to the player.
- Implement a polling mechanism to check if the opponent has also submitted their hash.

#### Polling Endpoint:
```
GET https://retzark-engine.retzark.com/match/{matchId}
```

Check the response for an indication that both players have submitted their hashes.

### 2.6 Transitioning to Wagering Phase

Once both players have submitted their hashes:

- Update the UI to reflect that both players are ready.
- Display face-down cards in the opponent's card slots to visually indicate that they have made their selection.
- Prepare to transition to the wagering phase.
- Enable wagering controls and display relevant wagering information (e.g., current balance, minimum bet).

## 3. Wagering System Implementation

### 3.1 Overview

The wagering system allows players to place bets during the match. It includes various actions such as betting, checking, calling, raising, and folding.

### 3.2 Wagering Process

1. Players join the match and place their cards on the battlefield in the app.
2. Players submit their card hash to the Retzark engine.
3. The wagering phase begins with an automatic buy-in based on the match rank.
4. Players take turns wagering until the wagering phase concludes.
5. Players reveal their cards.

### 3.3 Wagering Actions and Criteria

#### 3.3.1 Bet
- **Criteria**: Available when it's the player's turn to wager and no previous bet has been made in the current round.
- **Action**: Player places the initial bet of a specific amount.

#### 3.3.2 Check
- **Criteria**: Available when it's the player's turn to wager and no previous bet has been made in the current round.
- **Action**: Player passes the turn without placing a bet.

#### 3.3.3 Call
- **Criteria**: Available when it's the player's turn to wager and there's an existing bet from the opponent.
- **Action**: Player matches the current bet amount.

#### 3.3.4 Raise
- **Criteria**: Available when it's the player's turn to wager and there's an existing bet from the opponent.
- **Action**: Player increases the current bet amount.

#### 3.3.5 Fold
- **Criteria**: Available at any time during the player's turn to wager.
- **Action**: Player forfeits the match.

### 3.4 POST Requests to Retzark Engine

#### 3.4.1 Bet

```http
POST https://retzark-engine.retzark.com/wager/bet
Content-Type: application/json

{
  "player": "string",
  "matchId": "string",
  "wagerAmount": number,
  "signature": "string"
}
```

#### 3.4.2 Check

```http
POST https://retzark-engine.retzark.com/wager/check
Content-Type: application/json

{
  "player": "string",
  "matchId": "string"
}
```

#### 3.4.3 Call

```http
POST https://retzark-engine.retzark.com/wager/call
Content-Type: application/json

{
  "matchId": "string",
  "player": "string",
  "signature": "string",
  "betId": "string"
}
```

#### 3.4.4 Raise

```http
POST https://retzark-engine.retzark.com/wager/raise
Content-Type: application/json

{
  "matchId": "string",
  "player": "string",
  "signature": "string",
  "betId": "string",
  "raiseAmount": number
}
```

#### 3.4.5 Fold

```http
POST https://retzark-engine.retzark.com/wager/fold
Content-Type: application/json

{
  "matchId": "string",
  "player": "string",
  "signature": "string",
  "betId": "string"
}
```

#### 3.4.6 Get Match Wager Details

```http
GET https://retzark-engine.retzark.com/wager/{matchId}
```

This endpoint provides details about the match wager, including the automatic buy-in amount based on the match rank.

### 3.5 Implementation Steps

1. **Initial Setup**
   - After joining the match, prompt players to place their cards on the battlefield in the app.
   - Once cards are placed, submit the card hash to the Retzark engine using the "Submit Card Hash" POST request.

2. **Start Wagering Phase**
   - Retrieve the match wager details, including the automatic buy-in amount, using the "Get Match Wager Details" endpoint.
   - Apply the automatic buy-in based on the match rank.
   - The player in the player1 position starts the wagering phase.
   - Display available actions based on the criteria outlined above.

3. **Player's Turn to Wager**
   - When it's the player's turn to wager, enable the appropriate action buttons based on the current game state.
   - Disable action buttons when it's not the player's turn to wager.

4. **Perform Wagering Action**
   - When a player selects an action, send the corresponding POST request to the Retzark engine.
   - Include the "signature" field in all wager requests. The signature should be the content of the post request without the signature, signed with the player's Hive key.
   - For "Call," "Raise," and "Fold" actions, include the "betId" field if a bet has been made.
   - Wait for the response from the engine before proceeding.

5. **Update UI**
   - Update the UI to reflect the current wager amounts and available actions after each turn.

6. **State Management**
   - Continuously check the wager status to determine if the wagering has completed or if it's the player's turn to wager.

7. **End Wagering Phase**
   - The wagering phase ends when both players check, one player calls the other's bet, or a player folds.
   - Once the wagering phase is complete, proceed to the card reveal phase.

## 4. Revealing Cards

After the wagering phase is complete, players reveal their cards.

### 4.1 Reveal Cards Request

To reveal the cards, send a POST request to the Retzark engine:

```http
POST https://retzark-engine.retzark.com/match/reveal/{matchId}
Content-Type: application/json

{
  "player": "string",
  "cards": [number]
}
```

- `matchId`: The ID of the current match.
- `player`: The username of the player revealing their cards.
- `cards`: An array of card IDs that the player selected.

### 4.2 Processing Reveal

- Send this request for each player to reveal their cards.
- Wait for confirmation from the Retzark engine that both players have revealed their cards.
- Once both players have revealed their cards, the game can proceed to the battle simulation phase.

## 5. Battle History Simulation

### 5.1 Overview

This section outlines the process of simulating the battle history received from the Retzark engine within the Retzark app. The simulation involves visualizing card movements, attacks, health updates, and base damage based on the battle history data received from the engine.

### 5.2 Battle History Structure

The battle history is obtained from the Get Match Details endpoint at the end of each round. The structure is as follows:

```json
{
  "matchId": "string",
  "round": number,
  "battleHistory": [
    {
      "attacker": {
        "player": "string",
        "cardId": "string",
        "position": number
      },
      "defender": {
        "player": "string",
        "cardId": "string",
        "position": number
      },
      "damage": number,
      "attackType": "string",
      "effects": [
        {
          "type": "string",
          "value": number
        }
      ],
      "cardRemoved": boolean
    }
  ],
  "baseHealth": {
    "player1": number,
    "player2": number
  },
  "remainingCards": {
    "player1": [
      {
        "cardId": "string",
        "position": number,
        "health": number
      }
    ],
    "player2": [
      {
        "cardId": "string",
        "position": number,
        "health": number
      }
    ]
  }
}
```

### 5.3 Simulation Process

1. **Fetch Battle History**
   - Retrieve the battle history from the Get Match Details endpoint after each round.
   - Use the following GET request to fetch the match details:
     ```
     GET https://retzark-engine.retzark.com/match/{matchId}
     ```
   - Parse the JSON response to extract the battleHistory array and other relevant data.

2. **Initialize Simulation**
   - Set up the initial game state based on the current round's starting positions.
   - Display the current base health for both players.

3. **Process Battle Events**
   - Iterate through each event in the battleHistory array.
   - For each event, perform the following steps:

     a. **Identify Attacker and Defender**
     - Locate the attacker and defender cards on the game board based on their player and position.

     b. **Animate Attack**
     - Move the attacker card towards the defender card.
     - Display appropriate visual effects for the attack type (e.g., projectile, melee swing).

     c. **Apply Damage**
     - Update the defender card's health based on the damage value.
     - Display a damage number animation over the defender card.

     d. **Handle Effects**
     - Apply any additional effects listed in the event (e.g., poison, burn, heal).
     - Display

     e. **Check for Card Removal**
     - If cardRemoved is true, animate the defender card's removal from the board.

     f. **Update Base Health**
     - If the attack was against a base (defender cardId is null), update and display the new base health.

4. **Final State Update**
   - After processing all events, update the game board to reflect the remainingCards data.
   - Ensure all card positions and health values match the data provided.

5. **Prepare for Next Round**
   - Clear any temporary effects or animations.
   - Update the round number display.

### 5.4 Key Components to Implement

1. **BattleHistoryManager**
   - Responsible for fetching and parsing the battle history data.
   - Coordinates the overall simulation process.

2. **CardAnimator**
   - Handles individual card animations for attacks, movements, and removals.

3. **EffectManager**
   - Applies and visualizes various effects on cards (e.g., poison, burn, heal).

4. **HealthDisplay**
   - Updates and displays health values for cards and bases.

5. **BoardStateManager**
   - Manages the current state of the game board, including card positions and presence.

### 5.5 Best Practices

1. **Timing Control**: Implement a system to control the speed of the simulation, allowing for pausing, slowing down, or speeding up the playback.

2. **Error Handling**: Gracefully handle any discrepancies between the expected and actual data received from the engine.

3. **Performance Optimization**: Use object pooling for frequently created and destroyed objects (e.g., damage number displays, effect particles).

4. **Flexible Positioning**: Design the system to handle variable numbers of card positions and potential future expansions.

5. **State Synchronization**: Regularly compare the simulated state with the provided remainingCards data to ensure accuracy.

6. **Accessibility**: Include options for color-blind modes and text descriptions of actions for accessibility.

## 6. Conclusion

This comprehensive technical documentation provides a detailed overview of the key processes involved in implementing the Retzark game. From matchmaking and game setup to the intricate battle simulation, developers can use this guide to understand and implement the various components of the game.

Key areas covered include:
- Matchmaking and initial game setup
- Card selection and hash submission process
- Wagering system implementation
- Card revealing mechanism
- Battle history simulation and visualization

By following these guidelines and best practices, developers can create a robust, engaging, and fair gaming experience for Retzark players. Remember to regularly update this documentation as the game evolves and new features are added.

## 7. Appendix

### 7.1 API Endpoints Summary

Here's a quick reference of all the API endpoints used in the Retzark game:

1. Join Waiting Room: `POST https://retzark-engine.retzark.com/match/joinWaitingRoom`
2. Get Player Status: `GET https://retzark-engine.retzark.com/player/{username}`
3. Get Match Details: `GET https://retzark-engine.retzark.com/match/{matchId}`
4. Submit Card Hash: `POST https://retzark-engine.retzark.com/match/submitCardsHash`
5. Bet: `POST https://retzark-engine.retzark.com/wager/bet`
6. Check: `POST https://retzark-engine.retzark.com/wager/check`
7. Call: `POST https://retzark-engine.retzark.com/wager/call`
8. Raise: `POST https://retzark-engine.retzark.com/wager/raise`
9. Fold: `POST https://retzark-engine.retzark.com/wager/fold`
10. Get Match Wager Details: `GET https://retzark-engine.retzark.com/wager/{matchId}`
11. Reveal Cards: `POST https://retzark-engine.retzark.com/match/reveal/{matchId}`

### 7.2 Glossary

- **Hive Blockchain**: The decentralized blockchain network on which Retzark operates.
- **Custom JSON**: A type of operation on the Hive blockchain used to interact with dApps.
- **Card Hash**: A cryptographic hash of the selected cards, used to ensure fair play.
- **Wagering**: The process of placing bets during a match.
- **Battle History**: A record of all actions and their outcomes during a match.
- **RET Tokens**: The in-game currency used for wagering in Retzark.
