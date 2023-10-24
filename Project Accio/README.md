# Project Accio
 A 1 v 1 Android game trying to simulate the classical "two wizards shooting a colorful death ray towards each other but they are both equally powerful so they end up in sort of an inverse tug of war"-trope. I wanted to do something cool with pattern matching and trying to utilize firebase for a real time duel game. 

 ## Pattern Matching

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



  
