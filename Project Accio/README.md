# Project Accio
 A 1 v 1 Android game trying to simulate the classical "two wizards shooting a colorful death ray towards each other but they are both equally powerful so they end up in sort of an inverse tug of war"-trope. I wanted to do something cool with pattern matching and trying to utilize firebase for a real time duel game. 

 This is a broad strokes overview of the core mechanics. For the full codebase visit [the reposistory](https://github.com/emilxf-0/project-accio)

 ## Pattern Matching

The core gameplay is pretty straightforward. You try to match the symbol as fast as possible and the player who is fastest gains momentum. 

The player gets momentum | The opposing player gets momentum
:------------------------|------------------------------------------:
![](https://github.com/emilxf-0/Portfolio/blob/main/Project%20Accio/PlayerMomentum.gif) |![](https://github.com/emilxf-0/Portfolio/blob/main/Project%20Accio/EnemyMomentum.gif)


Pattern matching with touch input proved to be a challenge. In order to properly determine if the pattern actually matched the predetermined shape I had to implement a  [Dynamic Time Warping Algorithm](https://en.wikipedia.org/wiki/Dynamic_time_warping). 

```c#
    private float Match()
    {
        float[] distanceRow = new float[preDefinedSymbolPoints.Length + 1];

        for (int j = 0; j <= preDefinedSymbolPoints.Length; j++)
        {
            distanceRow[j] = float.PositiveInfinity;
        }
        
        distanceRow[0] = 0;
        
        for (int i = 1; i <= userSymbolPoints.Length; i++)
        {
            float[] newDistanceRow = new float[preDefinedSymbolPoints.Length + 1];
            newDistanceRow[0] = float.PositiveInfinity;
            
            for (int j = 1; j <= preDefinedSymbolPoints.Length; j++)
            {
                float cost = Vector3.Distance(userInputPoints[i - 1], preDefinedSymbolPoints[j - 1]);
                newDistanceRow[j] = cost + Mathf.Min(distanceRow[j], Mathf.Min(newDistanceRow[j - 1], distanceRow[j - 1]));
            }

            distanceRow = newDistanceRow;
        }
        
        return distanceRow[preDefinedSymbolPoints.Length];
    }
```
This function returns a float that I use to compare the pattern the player makes with the predefined symbol. The magic numbers are there because I had to manually test the threshold values for different symbols on different devices (obviously an improvement to be made at some point).

  ```c#
    private bool CompareInputWithSymbol(GameManager.Symbols symbols)
    {
        var thresholdValue = Match();

        switch (symbols)
        {
            case GameManager.Symbols.TRIANGLE:
                if (thresholdValue is < 108 or > 150) return false;
                return true;

            case GameManager.Symbols.SQUARE:
                if (thresholdValue is < 200 or > 246) return false;
                return true;

            case GameManager.Symbols.PENTAGRAM:
                if (thresholdValue is < 265 or > 365) return false;
                return true;

            case GameManager.Symbols.LIGHTNING:
                if (thresholdValue is < 88 or > 118) return false;
                return true;

        }

        throw new ArgumentException("No match for: " + symbols);

    }
```

In the update method I handle the touch input and when the player lifts their finger the result is sent to the database via `SendPlayerInfo()`

```c#
    void Update()
    {
        currentSymbol = GameManager.Instance.sequence.currentSequenceItem;
        preDefinedSymbolPoints = new Vector3[preDefinedSymbol[(int)currentSymbol].positionCount];
        
        if (Input.touchCount > 0)
        {
            Touch touch = Input.GetTouch(0);

            if (touch.phase == TouchPhase.Moved)
            {
                particleSystem.Play();
                Vector3 screenPos = touch.position;
                screenPos.z = Camera.main.nearClipPlane;
                Vector3 touchWorldPos = Camera.main.ScreenToWorldPoint(screenPos);
                userInputPoints.Add(touchWorldPos);
            
                lineRenderer.positionCount = userInputPoints.Count;
                lineRenderer.SetPositions(userInputPoints.ToArray());
                
                particleSystem.transform.position = touchWorldPos;
            }

            if (touch.phase == TouchPhase.Ended)
            {
                particleSystem.Stop();
                userSymbolPoints = new Vector3[lineRenderer.positionCount];
                fadeStartTime = Time.time;
                startFade = true;

                GameManager.Instance.sequence.CompareInputWithSequence(CompareInputWithSymbol(currentSymbol));
               
                SendPlayerInfo();
            }

        }
        
        if (startFade == false)
        {
            return;
        }
        
        FadeLine();
    }
```

Which is a nice segue into the main assignment, which was to build a mobile game that in some form uses saved data stored in an online database (firebase).

## Firebase

So, how this works is that both players sends info about their move to Firebase

```c#
private void SendPlayerInfo()
    {
        var playerReaction = GameManager.Instance.GetPlayerTimeStamp();
        var playerID = GameManager.Instance.playerID;
        var sequencePosition = GameManager.Instance.sequence.sequencePosition;
        var gameSessionID = GameManager.gameSessionID;
        var createdCorrectSymbol = CompareInputWithSymbol(currentSymbol);

        
        if (DatabaseAPI.Instance.singlePlayerGame)
        {
            GameManager.Instance.latestPlayerTimestamp = playerReaction;
            GameManager.Instance.SinglePlayerGame();
        }
        else
        {
            DatabaseAPI.Instance.SendAction(new PlayerInfo(playerID, playerReaction, sequencePosition, gameSessionID, createdCorrectSymbol), () =>
            {
                // Action was sent!
            }, exception => { Debug.Log(exception); });
        }
    }
```

Then the GameManager listens for changes to the database

```c#
    private void Start()
    {
        DatabaseAPI.Instance.SetPlayerID();
        
        if (SceneManager.GetSceneByName("GamePlay").isLoaded)
        {
            DatabaseAPI.Instance.ListenForAction(InstantiateMomentum, Debug.Log);
        }
        
        DatabaseAPI.Instance.isListening = true;
    }
```

I check if the playerID from the latest message matches the player's playerID

```c#
   private void InstantiateMomentum(PlayerInfo playerInfo)
    {
        string enemyPlayerID = playerInfo.playerID;
        
        // Check if it's an enemy action or friendly action
        if (enemyPlayerID == playerID || enemyPlayerID == "0")
        {
            CheckIfPlayerShouldHaveMomentum();
            return;
        }
        
        // Set the last known enemy data here
        enemyReactionTime = playerInfo.playerReactionTime;
        enemySequencePosition = playerInfo.sequencePosition;
        enemyCreatedCorrectSymbol = playerInfo.createdCorrectSymbol;
        
        
        DecideWhoHasMomentum(enemyReactionTime, enemyCreatedCorrectSymbol, enemySequencePosition);
        lastEnemyTimestamp = enemyReactionTime;
        lastEnemyPosition = enemySequencePosition;
    }
```

Then check if the player should have the momentum

```c#
   public void CheckIfPlayerShouldHaveMomentum()
    {
        var playerSequencePosition = sequence.sequencePosition;

        if (gameHasStarted == false && sequence.inputMatchSequence == false)
            return;

        // If both players misses nothing happens
        if (enemyCreatedCorrectSymbol == false && sequence.inputMatchSequence == false)
        {
            return;
        }

        if (sequence.inputMatchSequence == false)
        {
            return;
        }
        
        gameHasStarted = true;
        
        if (sequence.inputMatchSequence && enemyCreatedCorrectSymbol == false)
        {
            // If player already have momentum increase damage dealt
            if (healthManager.enemyMomentum == false)
            {
                //healthManager.damage += 0.1f;
                return;
            }
            
            // If enemy have momentum decrease damage taken or gain momentum
            if (playerSequencePosition < enemySequencePosition)
            {
                //healthManager.damage -= 0.1f;
            }
            else
            {
                healthManager.enemyMomentum = true;
            }
        }

        if (sequence.inputMatchSequence && enemyCreatedCorrectSymbol)
        {
            // If player is behind set enemy momentum to true
            if (playerSequencePosition < enemySequencePosition)
            {
                healthManager.enemyMomentum = true;
            }
            else if (playerSequencePosition == enemySequencePosition)
            {
                if (latestPlayerTimestamp > enemyReactionTime)
                {
                    healthManager.enemyMomentum = true;
                }
                else if (enemyReactionTime > latestPlayerTimestamp && healthManager.enemyMomentum == false)
                {
                    //healthManager.damage += 0.1f;
                }
                else
                {
                    healthManager.enemyMomentum = false;
                }
            }
            else
            {
                healthManager.enemyMomentum = false;
            }
        }

        if (lastEnemyPosition < sequence.sequencePosition)
        {
            healthManager.enemyMomentum = false;
        }
    }
```

Because I have to make sure the action is mirrored on the other device I need one check for enemy momentum and one for player momentum

```c#
public void CheckIfEnemyShouldHaveMomentum(float enemyTimeStamp, bool enemySymbolIsCorrect, int enemyPosition)
    {
        var playerSequencePosition = sequence.sequencePosition;
        
        if (gameHasStarted == false && enemySymbolIsCorrect == false)
            return;

        // If both players misses nothing happens
        if (enemySymbolIsCorrect == false && sequence.inputMatchSequence == false)
        {
            return;
        }

        if (enemySymbolIsCorrect == false)
        {
            return;
        }
        
        gameHasStarted = true;
        
        if (enemySymbolIsCorrect && sequence.inputMatchSequence == false)
        {
            // If enemy already have momentum increase damage taken
            if (healthManager.enemyMomentum)
            {
                //healthManager.damage += 0.1f;
                return;
            }
            
            // If enemy doesn't have momentum decrease damage dealt or lose momentum
            if (enemyPosition < playerSequencePosition)
            {
                //healthManager.damage -= 0.1f;
            }
            else
            {
                healthManager.enemyMomentum = true;
            }
        }

        if (enemySymbolIsCorrect && sequence.inputMatchSequence)
        {
            // If enemy is behind set momentum to false
            if (enemyPosition < playerSequencePosition)
            {
                healthManager.enemyMomentum = false;
            }
            else if (enemyPosition == playerSequencePosition)
            {
                if (enemyTimeStamp > latestPlayerTimestamp)
                {
                    healthManager.enemyMomentum = false;
                }
                else if (enemyTimeStamp < latestPlayerTimestamp && healthManager.enemyMomentum)
                {
                    //healthManager.damage += 0.1f;
                }
                else
                {
                    healthManager.enemyMomentum = true;
                }
            }
            else
            {
                healthManager.enemyMomentum = true;
            }
        }
        
    }
```


```c#

```
