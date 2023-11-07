# HeishaMonRules

## yada

yada yada

### on System#Boot

The first section of your ruleset. Here you can confiugre some settings and are the global variables defined.

<details>

<summary>on System#Boot</summary>

```LUA
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

### syncOpenTherm

This function synchronizes the heatpump values with your OpenTherm thermostat and back.

<details>

<summary>syncOpenTherm</summary>

```LUA
on syncOpenTherm then
    if #allowSyncOT == 1 then
        ?outletTemp = @Main_Outlet_Temp;
        ?inletTemp = @Main_Inlet_Temp;
        ?outsideTemp = @Outside_Temp;
        ?dhwTemp = @DHW_Temp;
        ?dhwSetpoint = @DHW_Target_Temp;
        if ?chEnable == 1 then
            #chEnable = 1;
        else
            #chEnable = 0;
        end
    end
    #dhwEnable = ?dhwEnable;
    if #maxTa != -1 then
        ?maxTSet = #maxTa;
    end
    if @Compressor_Freq == 0 then
        ?flameState = 0;
        ?chState = 0;
        ?dhwState = 0;
    else
        ?flameState = 1;
        if @ThreeWay_Valve_State == 0 then
            ?chState = 1;
            ?dhwState = 0;
        else
            ?chState = 0;
            ?dhwState = 1;
        end
    end
end
```

</details>

### calcHeatCurve

This function calculates the current target temperature based on the configured Heat Curve.

<details>

<summary>calcHeatCurve</summary>

```LUA
on calcHeatCurve then
    if #allowCalcHeatCurve == 1 then
        if isset(@Z1_Heat_Curve_Target_Low_Temp) && isset(@Z1_Heat_Curve_Outside_High_Temp) && isset(@Z1_Heat_Curve_Target_High_Temp) && isset(@Z1_Heat_Curve_Outside_Low_Temp) && isset(@Outside_Temp) then
            $Ta1 = @Z1_Heat_Curve_Target_Low_Temp;
            $Tb1 = @Z1_Heat_Curve_Outside_High_Temp;
            $Ta2 = @Z1_Heat_Curve_Target_High_Temp;
            $Tb2 = @Z1_Heat_Curve_Outside_Low_Temp;
            $Tb3 = @Outside_Temp;
            if $Tb3 >= $Tb1 then
                #maxTa = $Ta1;
            else
                if $Tb3 <= $Tb2 then
                    #maxTa = $Ta2;
                else
                    #maxTa = 1 + floor(0.9 + $Ta1 + (($Tb1 - $Tb3) * ($Ta2 - $Ta1) / ($Tb1 - $Tb2)));
                end
            end
        end
    end
end
```

</details>