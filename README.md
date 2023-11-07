# HeishaMonRules

## yada

yada yada

### on System#Boot

The first section of your ruleset. Here you can confiugre some settings and are the global variables defined.

<details>

<summary>on System#Boot</summary>

```
on System#Boot then
    #allowSyncOT = 1;
    #allowCalcHeatCurve = 1;

    #chEnable = -1;
    #maxTa = -1;
    setTimer(1,10);
end

on timer=1 then
    syncOpenTherm();
    calcHeatCurve();
    setTimer(1,15);
end
```

</details>