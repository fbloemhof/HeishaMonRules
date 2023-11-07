# HeishaMon Rules

## Some introduction

Here you find a collection of different rules to use with the [HeishaMonitor](https://github.com/Egyras/HeishaMon). 

Special thanks to [@CurlyMoo](https://github.com/CurlyMoo) and [@blb4github](https://github.com/blb4github). Most of the rules are inspired by their work.

> [!WARNING]  
> Usage of the rules is at your own risk.

## Instructions

You can pick and mix the functions you like. To start you always need the [`System#Boot`](#on-systemboot) section, this is where the basics are configured.

### on System#Boot

The first section of your ruleset. Here you can confiugre some settings and are the global variables defined. Default all functions are disabled (the `#allow` variabled), you can enable them by setting `0` to `1`.

The second part of the global variables are mostly helpers. Where possible the are defaulted to `-1` so you can monitor the results of the functions easily.

At the end the first timer, `1`, is set to run after 10 seconds. This timer is also set in this block. This is our global timer that recurs every 30 seconds to hendle all the main functions.

<details>

<summary>System#Boot</summary>

```LUA
on System#Boot then
    #allowSyncOT = 0;
    #allowCalcHeatCurve = 0;
    #allowSetQuietMode = 0;
    #allowshiftOnOpenTherm = 0;

    #chEnable = -1;
    #maxTa = -1;
    #quietModeHelper = 1;
    #quietModePrevious = -1;
    #chEnableCntr = -1;
    #chDisableCntr = -1;
    #chShiftHelper = 2;
    setTimer(1,10);
end

on timer=1 then
    calcHeatCurve();
    syncOpenTherm();
    setQuietMode();
    shiftOnOpenTherm();
    setTimer(1,30);
end
```

</details>

### syncOpenTherm

This function synchronizes the heatpump values with your OpenTherm thermostat.

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

This function calculates the current target temperature based on the configured Heat Curve When using `syncOpenTherm` the `#maxTa` value will be synced to the thermostat as `?maxTSet`.

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

### setQuietMode

This function sets the quiet mode based on a combination of the current time and the value of `@Outside_Temp`. It will help the heatpump running more stable and in low frequencies. 

> [!IMPORTANT]  
> Using this function will enable quiet mode which might impact your power usage and the performance of your heatpump.

<details>

<summary>setQuietMode</summary>

```LUA
on timer=2 then
    #quietModeHelper = 1;
    #quietMode = 0;
end

on setQuietMode then
    if #allowSetQuietMode == 1 then
        if isset(@Outside_Temp) && isset(@Heatpump_State) then
            if #quietModeHelper == 1 then
                if @Outside_Temp < 2 then
                    if %hour > 22 || %hour < 7 then
                        #quietMode = 1;
                    else
                        #quietMode = 0;
                    end
                end
                if @Outside_Temp < 5 then
                    #quietMode = 1;
                end
                if @Outside_Temp < 10 then
                    #quietMode = 2;
                else
                    #quietMode = 3;
                end
                if #quietModePrevious != #quietMode && @Heatpump_State == 1 then
                    setTimer(2, 900);
                    #quietModeHelper = 0;
                    #quietModePrevious = #quietMode;
                    @SetQuietMode = #quietMode;
                end
            end
        end
    end
end
```

</details>

### shiftOnOpenTherm

> [!WARNING]  
> This function is still in experimental phase.

This function shifts the request temperature up based on the chEnable status of the thermostat. If the thermostat is requesting heat for longer then 30 minutes, the `@SetZ1HeatRequestTemperature` is shifted to the value set in `#chShiftHelper` (see the [`System#Boot`](#on-systemboot) section) to speed up your warm up process. If the thermostat doesn't request any heat for longer then 30 minutes, the `@SetZ1HeatRequestTemperature` is set back to 0.

<details>

<summary>shiftOnOpenTherm</summary>

```LUA
on shiftOnOpenTherm then
    if #allowshiftOnOpenTherm == 1 then
        if ?chEnable == 0 && ?chSetpoint == 10 then
            #chDisableCntr = #chDisableCntr + 1;
        end
        if ?chEnable == 1 && ?chSetpoint != 10 then
            #chEnableCntr = #chEnableCntr + 1;
        end
        if #chEnableCntr == 60 then
            if #chShiftHelper != 2 then
                #chShiftHelper = 2;
                @SetZ1HeatRequestTemperature = 2;
            end
            #chEnableCntr = 0;
        end
        if #chDisableCntr == 60 then
            if #chShiftHelper != 0 then
                #chShiftHelper = 0;
                @SetZ1HeatRequestTemperature = 0;
            end
            #chDisableCntr = 0;
        end
    end
end
```

</details>