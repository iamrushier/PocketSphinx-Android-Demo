// Code here
package com.example.numberrecognizer;

import android.Manifest;
import android.app.Activity;
import android.content.pm.PackageManager;
import android.os.AsyncTask;
import android.os.Bundle;

import androidx.annotation.NonNull;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

import android.view.View;
import android.widget.Button;
import android.widget.TextView;
import android.widget.Toast;

import java.io.File;
import java.io.IOException;
import java.lang.ref.WeakReference;
import java.util.HashMap;

import edu.cmu.pocketsphinx.Assets;
import edu.cmu.pocketsphinx.Hypothesis;
import edu.cmu.pocketsphinx.RecognitionListener;
import edu.cmu.pocketsphinx.SpeechRecognizer;
import edu.cmu.pocketsphinx.SpeechRecognizerSetup;

import static android.widget.Toast.makeText;

public class MainActivity extends Activity implements RecognitionListener {

    private SpeechRecognizer recognizer;
    private TextView recognizedTextView;
    private Button startButton;
    private Button clearButton;
    private static final String KWS_SEARCH = "wakeup"; // Not used in this simplified version
    private static final String DIGITS_SEARCH = "digits";
    private static final int PERMISSIONS_REQUEST_RECORD_AUDIO = 1;
    private boolean isListening = false; // Track listening state
    private HashMap<String, Integer> captions;

    @Override
    public void onCreate(Bundle state) {
        super.onCreate(state);
        setContentView(R.layout.activity_main);

        recognizedTextView = findViewById(R.id.recognized_text);
        startButton = findViewById(R.id.start_button);
        clearButton = findViewById(R.id.clear_button);

        int permissionCheck = ContextCompat.checkSelfPermission(getApplicationContext(), Manifest.permission.RECORD_AUDIO);
        if (permissionCheck != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.RECORD_AUDIO}, PERMISSIONS_REQUEST_RECORD_AUDIO);
            return;
        }

        new SetupTask(this).execute();

        startButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (recognizer != null) {
                    if (!isListening) {
                        recognizedTextView.setText(recognizedTextView.getText() + "(1)");
                        startVoiceRecognition();
                    } else {
                        recognizedTextView.setText(recognizedTextView.getText() + "(2)");
                        stopVoiceRecognition();
                    }
                    updateButtonText();
                }
            }
        });

        clearButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                recognizedTextView.setText("");
            }
        });
    }

    private static class SetupTask extends AsyncTask<Void, Void, Exception> {
        WeakReference<MainActivity> activityReference;

        SetupTask(MainActivity activity) {
            this.activityReference = new WeakReference<>(activity);
        }

        @Override
        protected Exception doInBackground(Void... params) {
            try {
                Assets assets = new Assets(activityReference.get());
                File assetDir = assets.syncAssets();
                activityReference.get().setupRecognizer(assetDir);
            } catch (IOException e) {
                return e;
            }
            return null;
        }

        @Override
        protected void onPostExecute(Exception result) {
            if (result != null) {
                ((TextView) activityReference.get().findViewById(R.id.recognized_text)).setText("Failed to init recognizer " + result);
            } else {
                //Recognizer is now ready. Set initial button state.
                activityReference.get().updateButtonText();
            }
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        if (requestCode == PERMISSIONS_REQUEST_RECORD_AUDIO) {
            if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                new SetupTask(this).execute();
            } else {
                finish();
            }
        }
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        if (recognizer != null) {
            recognizer.cancel();
            recognizer.shutdown();
        }
    }

    @Override
    public void onPartialResult(Hypothesis hypothesis) {
        if (hypothesis != null) {
            String text = hypothesis.getHypstr();
            recognizedTextView.setText(text);
        }
    }

    @Override
    public void onResult(Hypothesis hypothesis) {
        if (hypothesis != null) {
            String text = hypothesis.getHypstr();
            recognizedTextView.setText(text);
            Toast.makeText(getApplicationContext(), text, Toast.LENGTH_SHORT).show(); // Optional toast
        }
        stopVoiceRecognition();
        updateButtonText();
    }

    @Override
    public void onBeginningOfSpeech() {
        isListening = true;
    }

    @Override
    public void onEndOfSpeech() {
        isListening = false;
    }

    private void setupRecognizer(File assetsDir) throws IOException {
        recognizer = SpeechRecognizerSetup.defaultSetup()
                .setAcousticModel(new File(assetsDir, "en-us-ptm"))
                .setDictionary(new File(assetsDir, "cmudict-en-us.dict"))
                .getRecognizer();
        recognizer.addListener(this);
        File numbersGrammar = new File(assetsDir, "digits.gram");
        recognizer.addGrammarSearch(KWS_SEARCH, numbersGrammar); // Use numbers grammar for keyword search
    }

    @Override
    public void onError(Exception error) {
        recognizedTextView.setText(error.getMessage());
    }

    @Override
    public void onTimeout() {
        stopVoiceRecognition();
        updateButtonText();
    }

    private void startVoiceRecognition() {
        synchronized (this) {
            isListening = true;
            recognizer.startListening(KWS_SEARCH);
        }
    }

    private void stopVoiceRecognition() {

        synchronized (this) {
            recognizer.stop();
            isListening = false;
        }
    }

    private void updateButtonText() {
        if (isListening) {
            startButton.setText("Stop");
        } else {
            startButton.setText("Start");
        }
    }
}