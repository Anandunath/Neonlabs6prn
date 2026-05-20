```
# Neonlabs6prn

QPSK

clear; close all;
N = 1e6;
EbNodB = 0:10;
for i = 1:length(EbNodB)
% QPSK symbols: Generates a BPSK signal for both Real and Imaginary axes
s = (2*(randi([0 1],1,N))-1) + 1j*(2*(randi([0 1],1,N))-1);
% AWGN noise: Generates random noise for both Real and Imaginary axes
w = (randn(1,N) + 1j*randn(1,N))/sqrt(2*10^(EbNodB(i)/10));
r = s + w; % Received
% Simulated BER: Checks mistakes on Real and Imaginary bits together
simBER(i) = mean([real(s) ~= sign(real(r)), imag(s) ~= sign(imag(r))]);
end
theBER = 0.5*erfc(sqrt(10.^(EbNodB/10))); % Theoretical BER
semilogy(EbNodB, simBER, '-', EbNodB, theBER, 'o');
grid on;
xlabel('Eb/No (dB)');
ylabel('BER');
legend('Simulation', 'Theory');
title('QPSK BER');


BPSK

clear;
close all;
N=1e6;
EbNodB=0:10;
for i = 1:length(EbNodB) ;
s=2*(randi([0 1], 1 ,N))-1;
w=randn(1,N)/sqrt(2*10^(EbNodB(i)/10));
r=s+w;
simBER(i)=mean(s~=sign(r));
end
theBER=0.5*erfc(sqrt(10.^(EbNodB/10)));
semilogy(EbNodB , simBER , '-' , EbNodB , theBER , 'o');
grid on;
xlabel('Eb');
ylabel('ber');
legend('simulation');
title('bpsk');


Pulse shaping(exp 2)

% Parameters
N = 1000; M = 8; R = 1e6; SNRdB = 10;
t = (0:N-1)/(R*M); rolloff = 0.25;
sym = 2*randi([0 1],1,N/M)-1;
h = rcosdesign(rolloff,6,M,'sqrt');
x = conv(upsample(sym,M),h,'same');
np = 10^(-SNRdB/10)*var(x);
rx = x + sqrt(np/2)*(randn(size(x))+1i*randn(size(x)));
y = conv(rx,fliplr(conj(h)),'same');
est = y(1:M:end);
BER = mean(sym ~= sign(real(est)));
fprintf('BER = %.4e\n',BER);
subplot(3,1,1); plot(t,real(x)); title('Pulse Shaped');
subplot(3,1,2); plot(t,real(rx)); title('Received');
subplot(3,1,3); plot(t(1:M:end),real(est)); title('Estimated');



EYE DIAGRAM(exp 3)

N = 2000; M = 16; Fs = 1e6; Rb = 1e5; sps = Fs/Rb;
alpha = 0.5; span = 6; SNR = 20;
data = randi([0 M-1], N, 1);
symbols = qammod(data, M, 'UnitAveragePower', true);
tx = upsample(symbols, sps);
rrcos = rcosdesign(alpha, span, sps);
pulse = conv(tx, rrcos, 'same');
rx = awgn(pulse, SNR, 'measured');
mf = conv(rx, fliplr(rrcos), 'same');
eyediagram(mf, 2*sps);
scatterplot(mf(1:sps:end));
demod = qamdemod(mf(1:sps:end), M, 'UnitAveragePower', true);
BER = sum(demod ~= data)/N;
disp(['BER: ' num2str(BER)]);


% Pulse Shaping and Matched Filtering Simulation

clc;
clear;
close all;
num_symbols = 125; % Number of bits/symbols to transmit
oversampling_rate = 8; % Number of samples per symbol
num_samples = num_symbols * oversampling_rate; % Total samples (1000)
data_bits = round(rand(1, num_symbols));
symbols = 2 * data_bits - 1;
rolloff = 0.25; % Roll-off factor for the filter
filter_span = 6; % Filter length in symbols
h = rcosdesign(rolloff, filter_span, oversampling_rate, 'sqrt');
upsampled_symbols = upsample(symbols, oversampling_rate);
tx_signal = conv(upsampled_symbols, h, 'same');
SNR_dB = 10; % Signal-to-Noise Ratio
rx_signal = awgn(tx_signal, SNR_dB, 'measured');
matched_signal = conv(rx_signal, h, 'same');
estimated_symbols = matched_signal(1:oversampling_rate:end);
received_bits = estimated_symbols > 0;
errors = sum(data_bits ~= received_bits);
BER = errors / num_symbols;
fprintf('Bit Error Rate (BER): %.4f\n', BER);
t = 1:num_samples; % Simple time axis for plotting
figure('Name', 'Pulse Shaping and Matched Filtering', 'NumberTitle', 'off');
subplot(3,1,1);
plot(t, tx_signal, 'b', 'LineWidth', 1.5);
title('Transmitted Signal (After Pulse Shaping)');
xlabel('Sample Index'); ylabel('Amplitude');
grid on;
subplot(3,1,2);
plot(t, rx_signal, 'r');
title(['Received Signal with AWGN (SNR = ', num2str(SNR_dB), ' dB)']);
xlabel('Sample Index'); ylabel('Amplitude');
grid on;
subplot(3,1,3);
plot(t, matched_signal, 'g', 'LineWidth', 1.5); hold on;
stem(1:oversampling_rate:num_samples, estimated_symbols, 'k', 'Filled');
title('After Matched Filtering (Black stems are sampling points)');
xlabel('Sample Index'); ylabel('Amplitude');
legend('Filtered Signal', 'Sampled Peaks');
grid on;


```
