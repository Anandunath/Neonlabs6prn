```
# Neonlabs6prn

QPSK

clear ;close all;
num_bit=1e6;
EbNodB=0:1:10;
EbNo=10.^(EbNodB/10);
sim_BER=zeros(1,length(EbNodB));
for n=1:length(EbNodB)
   si=2*(round(rand(1,num_bit))-0.5);
   sq=2*(round(rand(1,num_bit))-0.5);
   s=si+1j*sq;
w=(1/sqrt(2*EbNo(n)))*(randn(1,num_bit)+1j*randn(1,num_bit));
   r=s+w;
   si_=sign(real(r));
   sq_=sign(imag(r));
   ber1=(num_bit-sum(si==si_))/num_bit;
   ber2=(num_bit-sum(sq==sq_))/num_bit;
   sim_BER(n)=mean([ber1 ber2]);
end
the_Ber = 0.5*erfc(sqrt(10.^(EbNodB/10)));
semilogy(EbNodB, sim_BER,'-');
hold on
semilogy(EbNodB,the_Ber,'ko');
title('BER curve for QPSK modulation');
legend('Simulation','Theoretical');
xlabel('EbNo(dB)')
ylabel('BER')
grid on



BPSK

clear ;close all;
num_bit=1e6;
EbNodB=0:1:10;
for i=1:length(EbNodB);
   s=2*(round(rand(1,num_bit))-0.5);
   w=(1/sqrt(2*10^(EbNodB(i)/10)))*randn(1,num_bit);
   r=s+w;
   s_est=sign(r);
   sim_BER(i)=(num_bit-sum(s==s_est))/num_bit;
end
the_Ber =0.5*erfc(sqrt(10.^(EbNodB/10)));
semilogy(EbNodB, sim_BER,'-');
hold on
semilogy(EbNodB,the_Ber,'ko');
title('BER curve for BPSK modulation');
legend('Simulation','Theoretical');
xlabel('EbNo(dB)')
ylabel('BER')
grid on



Pulse shaping and matched filter

N = 1000;
symbol_rate = 1e6;
oversampling_rate = 8;
sample_rate = symbol_rate * oversampling_rate;
t = (0:N-1)/sample_rate;
symbols = 2*round(rand(1, N/oversampling_rate))-1;
rolloff = 0.25;
h = rcosdesign(rolloff, 6, oversampling_rate, 'sqrt');
x = upsample(symbols, oversampling_rate);
x_shaped = conv(x, h, 'same');
SNR_dB = 10;
noise_power = 10^(-SNR_dB/10) * var(x_shaped);
noise = sqrt(noise_power/2) * (randn(size(x_shaped)) + 1i*randn(size(x_shaped)));
received_signal = x_shaped + noise;
matched_filter = conj(fliplr(h));
y = conv(received_signal, matched_filter, 'same');
estimated_symbols = y(1:oversampling_rate:end);
figure;
subplot(3,1,1);
plot(t, real(x_shaped));
title('Pulse Shaped Signal');
xlabel('Time (s)'); ylabel('Amplitude');
subplot(3,1,2);
plot(t, real(received_signal));
title('Received Signal with Noise');
xlabel('Time (s)'); ylabel('Amplitude');
subplot(3,1,3);
plot(t(1:oversampling_rate:end), real(estimated_symbols));
title('Estimated Symbols after Matched Filtering');
xlabel('Time (s)');
ylabel('Amplitude');
original_symbols = symbols;
errors = sum(original_symbols ~= sign(real(estimated_symbols)));
BER = errors / length(symbols);
fprintf('Bit Error Rate: %.4e\n', BER);
Bit Error Rate: 0.0000e+00



EYE DIAGRAM

N = 2000;
M = 16;
Fs = 1e6;
Rb = 1e5;
alpha = 0.5;
span = 6;
sps = Fs/Rb;
data = randi([0, M-1], N, 1);
symbols = qammod(data, M, 'UnitAveragePower', true);
tx_signal = zeros(N*sps,1);
tx_signal(1:sps:end) = symbols;
rrcos = rcosdesign(alpha, span, sps);
pulse_shaped = conv(tx_signal, rrcos, 'same');
SNR = 20;
noisy_signal = awgn(pulse_shaped, SNR, 'measured');
matched_signal = conv(noisy_signal, fliplr(rrcos), 'same');
figure;
eyediagram(matched_signal, 2*sps);
received_symbols = matched_signal(1:sps:end);
figure;
scatterplot(received_symbols);
title('Received Constellation');
demodulated_data = qamdemod(received_symbols, M, 'UnitAveragePower', true);
errors = sum(demodulated_data ~= data);
BER = errors/N;
disp(['Bit Error Rate (BER): ', num2str(BER)]);



 Performance of Waveform Coding Using PCM 

clear;close all;
time = 0:.0005:.05;
freq_msg=100;
dc_ofst=2;
signal=sin(2*pi*freq_msg*time)+dc_ofst;
figure;plot(time,signal);
xlabel('time');
ylabel('Amplitude');
title('Signal');
freq_sample=15*freq_msg;
samp_time=0:1/freq_sample:0.05;
samp_signal=dc_ofst+sin(2*pi*freq_msg*samp_time);
hold on;
plot(samp_time,samp_signal,'rx');
title('Sampled Signal');
legend('Original signal','Sampled signal');
L=8;
smin=round(min(signal));
smax=round(max(signal));
Quant_levl=linspace(smin,smax,L);
codebook = linspace(0,smax,L+1);
[index,quants] = quantiz(samp_signal,Quant_levl,codebook);
figure;plot(samp_time,samp_signal,'x',samp_time,quants,'.-');
title('Quantized Signal');
legend('Original signal','Quantized signal');
figure;plot(samp_time,index,'.-');
for i=1:length(index)
   bincode_sig{i}=dec2bin(round(index(i)),7);
end
disp('binary encoded signal');
disp(bincode_sig);
noise=quants-samp_signal;
figure;plot(samp_time,noise,'.-');
title('Noise');
r=snr(index,noise);
snr1=['SNR :',num2str(r)];
disp(snr1);


```
