# Tick

## Environement.GetTickCount() 

which gives you the number of ms since the start of the kernel.
It is a counter incremented by a hardware interrupt

## DateTime.Tick
"A single tick represents one hundred nanoseconds or one ten-millionth of a second."
(https://docs.microsoft.com/fr-fr/dotnet/api/system.datetime.ticks?view=netframework-4.7.2)

## The one in Stopwatch

correspond to the tick of the microprocessor and which depend on the instantaneous frequency of the CPU
On my desktop:   
Stopwatch.Frequency = 10 000 000
So a tick corresponds to 1/10 000 000 = 10^-7 second = 100 nanoseconds