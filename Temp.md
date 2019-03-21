# How to modelize event envelopping a bunch of fields

__Project:__ Caraibes

__Date:__ 3/21/2019

__Authors:__ Thomas PIERRAIN, Yi HAN


## Context

Till now, for this kind of requirement, our modelizaiton is for every field, we create a Intention<T>.  
From the Intention<T>.Intent(Changed/Delete/ToBeIgnored) in command, write side emit __A SINGLE__ event, envlopping all Intentions.   

__Problematic__:
We need meet new requirement to do "in place" changes (*french: mode à plat*).

Example, we have UsTaxFormsW8W9Needed is part of Umbrella > Distribution > Tax information.

| Date De Valeur | Value |
|----------------|-------|
| 2000           | true  |
| 2001           | false |
| 2005           | true  |
| 2019           | false |


 As "fellow fields", it has also:
- FlowThroughEntityByUsTaxLaw
- SubjectToFactaWithholdingTaxation
- FatcaStatusV2
    
Imagine that if we envelop all of these four fields in one event, we would always have to handle a nasty sitution: what if one field changed(__both value and dateDeValeur__), but not others?  

We have to copy all not changed values create new one with the existing dateDeValeur,  
Then, cancel the old event
Then, create new event with the not changes fields with ToBeIgnored intent; plus the new field with Change intent.

Too complicated!

## Solution

Split the "Whole functional zone changed" stuff to N events by field.

```CSharp
    public class UsTaxFormsW8W9NeededChanged : AggregateEvent
    {
        public bool? Value { get; }

        public IEnumerable<string> Tags { get; set; }

        public UsTaxFormsW8W9NeededChanged(Guid umbrellaId, bool? value, Timestamp dateDeValeur, Timestamp dateDeSaisie) : this(umbrellaId, value, new string[] { "umbrella.distribution.tax" }, dateDeValeur, dateDeSaisie)
        {
        }

        private UsTaxFormsW8W9NeededChanged(Guid umbrellaId, bool? value, IEnumerable<string> tags, Timestamp dateDeValeur, Timestamp dateDeSaisie) : base(umbrellaId, dateDeValeur, dateDeSaisie)
        {
            this.Value = value;
            this.Tags = tags;
        }
    }
```

__Benefits__:

1. We resolve the "in place modification"(*french: mode à plat*) problematic.  
    * When user changes only date de valeur: Patching date de valeur
    * When user changes only the field's value: Add new event
    * When user changes both the date de valeur and field's value: We cancel the old one then create a new one.

1. We resolve the problematic Milestones by Tag.
Till now, we use logical 'if' to group events to Milestone.   
Now that we have "Tags", we can categorize events by tag, then push the changes of event into different milestone type.  

1. We resolve the functional events grouping by Tag.

## Consequences
1. __TO BE STUDIED__:  In terms of Milestone, how to manage existing events without tags? Maintain the existing logic, and add addhoc logic to handle Tag? 

