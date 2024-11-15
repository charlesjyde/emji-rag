# Implementing a Regression Model for Time-To-Failure Prediction



% Task 3: Implementing a Regression Model for Time-To-Failure Prediction % Clear workspace and command window clear; clc; % Load the training data trainData = readtable('$train_$$selected.csv'); % Load the test data testData = readtable('$test_$$selected.csv'); % Extract features and target variable X = trainData{:, {'$s1$$', '$s2$$', '$s3$$', '$s4$$'}}; y = trainData.ttf; % Split the data into training and validation sets cv = cvpartition(size(X,1), 'HoldOut', 0.2); idx = cv.test; $X_$$train = X(~idx,:); $y_$$train = y(~idx); $X_$$val = X(idx,:); $y_$$val = y(idx); % Normalize the features (standardization) [$X_$$$train_$$norm, mu, sigma] = zscore($X_$$train); $X_$$$val_$$norm = ($X_$$val - mu) ./ sigma; % Define the kernel function for GPR kernelFunction = 'squaredexponential'; % Create and train the GPR model with specified PredictMethod gprMdl = fitrgp($X_$$$train_$$norm, $y_$$train, 'KernelFunction', kernelFunction, ...                 'Standardize', true, 'PredictMethod', 'exact'); % Predict on the validation set [$y_$$pred, $y_$$$pred_$$sd] = predict(gprMdl, $X_$$$val_$$norm); % Evaluate the model mae = mean(abs($y_$$val - $y_$$pred)); rmse = sqrt(mean(($y_$$val - $y_$$pred).^2)); fprintf('Validation MAE: %.2f\n', mae); Validation MAE: 34.04 fprintf('Validation RMSE: %.2f\n', rmse); Validation RMSE: 46.29 1

% Prepare test data $X_$$test = testData{:, {'$s1$$', '$s2$$', '$s3$$', '$s4$$'}}; $X_$$$test_$$norm = ($X_$$test - mu) ./ sigma; % Predict TTF for the test data [$ttf_$$predictions, $ttf_$$$pred_$$sd] = predict(gprMdl, $X_$$$test_$$norm); % Load the true TTF values for the test data $pm_$$truth = readtable('$PM_$$truth.txt', 'ReadVariableNames', false); $ttf_$$true = $pm_$$truth.$Var1$$; % Compare predictions with true values $test_$$mae = mean(abs($ttf_$$true - $ttf_$$predictions)); $test_$$rmse = sqrt(mean(($ttf_$$true - $ttf_$$predictions).^2)); fprintf('Test MAE: %.2f\n', $test_$$mae); Test MAE: 26.67 fprintf('Test RMSE: %.2f\n', $test_$$rmse); Test RMSE: 34.77 % Plot predicted vs true TTF figure('Name', 'Predicted vs True Time-To-Failure', 'NumberTitle', 'off'); plot(1:length($ttf_$$true), $ttf_$$true, 'o-', 'LineWidth', 1.5); hold on; plot(1:length($ttf_$$predictions), $ttf_$$predictions, 'x-', 'LineWidth', 1.5); % Calculate confidence intervals $upper_$$confidence = $ttf_$$predictions + 1.96 * $ttf_$$$pred_$$sd; $lower_$$confidence = $ttf_$$predictions - 1.96 * $ttf_$$$pred_$$sd; % Plot confidence intervals fill([1:length($ttf_$$predictions), fliplr(1:length($ttf_$$predictions))], ...      [$upper_$$confidence', fliplr($lower_$$confidence')], 'k', 'FaceAlpha', 0.1,  'EdgeColor', 'none'); title('Predicted vs True Time-To-Failure for Test Data'); xlabel('Engine Index'); ylabel('Time-To-Failure'); legend('True TTF', 'Predicted TTF', '95% Confidence Interval'); grid on; hold off; 2

3