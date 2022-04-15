# Asset maintenance planner (AMP)

AMP is the name of the algorithm that calculates the ideal time when to perform asset maintenance tasks within a given spatial region in Nijmegen city. First, it finds the best maintenance time for a given asset group (e.g. roads, vegetation, lighting, etc.) in a given sub-district. Next, it determines the best time point to perform maintenance over all asset groups given the individual asset group estimates. Finally, it suggests an optimal solution for a city district made up of multiple sub-districts, where all sub-district maintenance years are consecutive (if possible).

# Optimization algorithm: Sub-district level

## Step 1: Finding the optimal maintenance year within asset groups
Here, the algorithm simply computes the optimal maintenance year over all assets. It considers when each asset was last replaced (or built), and picks the next upcoming large maintenance task according to the corresponding asset template (note that we assume that all previously scheduled asset maintenance tasks have been performed).

## Step 2: Finding the optimal maintenance year between asset groups

The second step of the optimization algorithm aims to find the best possible year when

1. the largest number of asset groups (e.g. roads, plumbing, vegetation etc.) can be maintained and 
2. the individual asset group estimates for performing maintenance (from step 1) are as close to the solution as possible.

For this step there are several ways in which the user can influence the outcome. First, it is important that the user gives a range that specifies the possible maintenance years for each asset group, which also includes the optimal maintenance year from step 1. Consider the following example:

![image](https://user-images.githubusercontent.com/31699416/146896442-91c301f7-a194-4ba2-9493-24dcc998ede1.png)

As seen in the Figure, there are 9 different asset groups (Wegen, Riolering, openbare verlichting, Civiele kunstwerken, Groen, Verkeersregelinstallaties, Spelen, Afkoppelen aardgas, Woondeel). The grey disks denote the ideal year to perform maintenance for each asset group - the output of step 1. The encircled areas around the disks denote the time ranges that are also acceptable as specified by the asset manager. Finally, the blue line denotes the year that is currently selected by the asset manager, and to begin with, this is the year suggested by the algorithm.

In step 2a the algorithm first finds the time when most asset groups can be maintained at once, constrained by the time ranges (in the Figure you can see that only Wegen, Openbare verlichting, Civiele kunstwerken, Groen, Verkeersregelinstallaties and Woondeel are highlighted and included in the selection).

There are two ways in which the user can influence the algorithm at this step. First, it is possible to specify one leading asset group. This asset group has to be included in the solution (even if there are no other asset groups than can be maintained at the same time). Secondly, the user can specify a synergy matrix, which is a N x N matrix, with N denoting the number of asset groups. The row-column combination of each asset group pair contains their synergy. The synergy value is added to the total number of asset groups than can be maintained at a given time if both of the asset groups are available to be maintained at this time. By default this matrix is filled with zeros (no synergy). Consider the following example:

|              | Wegen    | Riool    | Groen    |
|--------------|----------|----------|----------|
| Wegen        | 0        | 2        | 0        |
| Riool        | 2        | 0        | 0        |
| Groen        | 0        | 0        | 0        |

In this example, `Wegen` and `Riool` have a synergy value of 2. That means that when we calculate the total number of asset groups that can be maintained at the same time, if both of these asset groups are included, we add +2 to the total value. When `Wegen` and `Riool` are included, but `Groen` is not this will yield a value of 4. On the other hand if `Groen` and `Riool` are included, but `Wegen` is not, this will yield a value of 2, since there is no synergy between `Riool` and `Groen`. The measure used to determine when most asset groups can be maintained at the same time in step 2a is therefore the total number of asset groups that can be maintained at the same time + any applicable synergies.

In step 2b, the algorithm finds the optimal time within the possible maintenance years when most asset groups can be maintained. Moreover, the user can specify a weights vector that weighs certain asset groups more heavily than others. For instance, if we want to obtain a result close to the optimal result of the `wegen` asset group, we could give this asset group a larger weight than the other asset groups. By default all asset groups are considered equal (all weights are equal to 1).

# Optimization algorithm: District level

Different city sub-districts (or neighborhoods) do not exist in isolation, they are part of a district (or borough), and share certain properties with neighboring sub-districts from the same district. For instance, if one district entry-point is closed due to maintenance, one should not also start maintenance work in a neighboring sub-district that would close another. To arguably mitigate the strain on the inhabitants of the neighborhood, and to make use of resources more effectively, it was chosen to try to perform maintenance work within a district in a phased fashion. That means, we try to perform asset maintenance in consecutive years over all sub-districts in a district.

This district level "phasing solution" builds on the optimization we already performed at the sub-district level. First, we set a hard constraint that all leading asset groups have to be included in the solution. This constraint supercedes the consecutiveness constraint. Hence, if there is no overlap between possible maintenance years of different leading asset groups of different sub-districts, it is likely that no solution with consecutive maintenance years exists. In that case we find the solution that has the fewest number of gap years.

While taking into account this leading asset group constraint we find the best district level solution by first selecting all solutions that fulfill the leading asset constraint and the consecutiveness constraint (if true consecutiveness is not possible due to the leading asset constraint we consider solutions that are as consecutive as possible). If there are multiple possible options, we find the optimal solution by following the same logic as in steps 2a and 2b of the sub-district level optimization. That is, we first find the candidate solutions where most asset groups can be maintained at the same time (+ any applicable synergies). Next, we minimize the mean squared error relative to the individual asset group estimates that are included in the candidate solution, applying any specified weights.
