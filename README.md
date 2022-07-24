# MATLAB fiber photometry analysis

data = TDTbin2mat('path/to/data/folder')

signal465=data.streams.x465A.data;  
signal405=data.streams.x405A.data;  
signal_fs=data.streams.x465A.fs;

#Normalize and plot FP traces
MeanFilterOrder = 1000; % for smoothing
MeanFilter = ones(MeanFilterOrder,1)/MeanFilterOrder;

fs_signal = 1:length(signal465);
sec_signal = fs_signal/signal_fs;  

reg = polyfit(signal405(60*signal_fs:end), signal465(60*signal_fs:end), 1);
a = reg(1);
b = reg(2);
controlFit = a.*signal405 + b;
controlFit = filtfilt(MeanFilter,1,double(controlFit));
normDat = (signal465 - controlFit)./controlFit;
delta465 = normDat * 100;     

#465 cube 1 data
figure
a = subplot(4,1,1);
plot(sec_signal(60*signal_fs:end), signal405(60*signal_fs:end));
title('raw control NE');
b = subplot(4,1,2);
plot(sec_signal(60*signal_fs:end), signal465(60*signal_fs:end));
title('raw signal');
c = subplot(4,1,3);
plot(sec_signal(60*signal_fs:end), signal465(60*signal_fs:end));
hold on
plot(sec_signal(60*signal_fs:end), controlFit(60*signal_fs:end));
title('fitted control');
d = subplot(4,1,4);
plot(sec_signal(60*signal_fs:end), delta465(60*signal_fs:end));
title('normalized signal');
linkaxes([a,b,c,d],'x');

#smoothing traces
delta465_filt = filtfilt(MeanFilter,1,double(delta465));

#detrend traces
delta465_filt_detr =detrend(delta465_filt);

#downsampling traces for plotting
ds_delta465_filt_detr = downsample(delta465_filt_detr, 100);
ds_sec_signal = downsample(sec_signal, 100);

figure
a = subplot(2,1,1);
plot(ds_sec_signal(60:end),ds_delta465_filt_detr(60:end))
title('465');
ylabel('dF/F')
xlabel('Time (s)')
