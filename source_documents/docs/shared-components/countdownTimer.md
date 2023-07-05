---
title: CountdownTimer
navTitle: CountdownTimer
---

#  CountdownTimer

## Description:

CountdownTimer adds a timer view to the display, that counts down to a predetermined end time.

<img src="images/CountdownTimer1.png" width="300" />

*CountdownTimer*


## Overview

#### CountdownTimer

Here we set up Timer clock for `CountdownTimer`. Setting up all the required information like start time, end time, and sunset time that would be needed to create a timer.

###### Parameters:

- `startTime:` Date
  - Time at which the timer clock will start
- `endTime:` Date
  - Time at which the timer clock will stop
- `sunsetTime:` Date
  - Time at which the timer clock will expire


#### CountdownTimerViewModel

Here we set up view model for `CountdownTimer`. We have delegates here which will be triggered whenever the state or input components changes in `CountdownTimer`.

###### Parameters:

- `startTime:` Date
  - UTC time when timer starts
- `endTime:` Date
  - UTC time when timer stops
- `sunsetTime:` Date
  - UTC time when timer expires
- `backgroundColor:` UIColor
  - Background color of TimerView
- `timerComponentTitleColor:` UIColor
  - Color for component titles (day, hour, minute, second)
- `timerComponentValueColor:` UIColor
  - Color for component values (03:21:14:09)
- `timerComponentTitleFont:` LDFont
  - Font for component titles (day, hour, minute, second)
- `timerComponentValueFont:` LDFont
  - Font for component values (03:21:14:09)
- `timerComponentTitleHighlightedFont?:` LDFont?
  - Font for biggest timer component title. (Default value is timerComponentTitleFont)
- `timerComponentValueHighlightedFont?:` LDFont?
  - Font for biggest timer component value. (Default value is timerComponentValueFont)
- `timerValueComponentSpacing:` CGFloat
  - Spacing for Timer value labels. (Default value is 3.0)
- `shouldHideTitleSection:` Bool
  - Boolean indicating whether we need to hide title section of the timer. (Default value is false)

###### Public methods available:

- startTimer()
  - This method is called when the timer is about to start by giving some information like `startTime, endTime, sunsetTime`
- cancelTimer()
  - This method is called when the timer has stopped by setting the timer components to `nil`


<img src="images/CountdownTimer2.png" width="320" />

*Example: CountdownTimer being used in Early Access Banner*


###### Possible states for `CountdownTimer`:
- started
  - If timer startTime has started
- anHourRemaining
  - If time remaining time is less than one hour. This helps to inform customers
- stopped
  - If endTime is earlier than currentDate, then we stop the timer
- expired
  - If the sunsetTime is earlier than the CurrentDate, then we disable the timer
- default


> `CountdownTimerViewModelDelegate` will be triggered whenever the state, input components changes in CountdownTimer. Below are some functions that are used:

- `func didChangedCountdownTimerState(_ state: CountdownTimerState)`
  - This method is invoked whenever countdownTimerState changes
- `func didChangedTimerComponents(_ timerComponents: DateComponents?)`
  - This method will be invoked whenever timer component changes (date, time, title)
- `func announceTimerDetails()`
  - This method will be invoked every 5 mins, announcing details for CountdownTimer


#### CountdownTimerView

Here we set up the view for `CountdownTimer`. UIStackView is used here to arrange the timer component details (days, hrs, mins, secs) and values (03:21:14:09). We use arrays to store valueLabels and timerLabels for `CountdownTimer`.

###### Arrays:
- timerTitleLabels = [LDLabel] ()
  - Array containing all the timer title labels
- timerValueLabels = [LDLabel] ()
  - Array containing all the timer value labels
