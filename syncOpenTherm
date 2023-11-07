on System#Boot then
    #allowSyncOT = 1;

    #chEnable = -1;
    #maxTa = -1;
    setTimer(1,10);
end

on timer=1 then
 syncOpenTherm();
 setTimer(1,15);
end

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