# huellaDigital

import static androidx.biometric.BiometricManager.Authenticators.BIOMETRIC_STRONG;
import static androidx.biometric.BiometricManager.Authenticators.DEVICE_CREDENTIAL;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.biometric.BiometricManager;
import androidx.biometric.BiometricPrompt;
import androidx.core.content.ContextCompat;
import android.content.Intent;
import android.os.Bundle;
import android.provider.Settings;
import android.util.Log;
import android.widget.Button;
import android.widget.ImageView;
import android.widget.Toast;

import java.util.concurrent.Executor;

public class MainActivity extends AppCompatActivity {

    private static final int REQUEST_CODE = 101010;

    ImageView imagenLogin;
    Button botonLogin;

    private Executor executor;
    private BiometricPrompt biometricPrompt;
    private BiometricPrompt.PromptInfo promptInfo;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        imagenLogin = findViewById(R.id.imageViewHuella);
        botonLogin = findViewById(R.id.BotonHuella);

        //Codigo de la pagina oficial
        BiometricManager biometricManager = BiometricManager.from(this);
        switch (biometricManager.canAuthenticate(BIOMETRIC_STRONG | DEVICE_CREDENTIAL)) {
            case BiometricManager.BIOMETRIC_SUCCESS:
                Log.d("MY_APP_TAG", "App can authenticate using biometrics.");
                Toast.makeText(this, "Se detectó el uso de biometría", Toast.LENGTH_SHORT).show();
                break;
            case BiometricManager.BIOMETRIC_ERROR_NO_HARDWARE:
                Log.e("MY_APP_TAG", "No biometric features available on this device.");
                Toast.makeText(this, "No se detectó ningun lector biometría", Toast.LENGTH_SHORT).show();
                break;
            case BiometricManager.BIOMETRIC_ERROR_HW_UNAVAILABLE:
                Log.e("MY_APP_TAG", "Biometric features are currently unavailable.");
                Toast.makeText(this, "Error al detectar el sensor de biometria", Toast.LENGTH_SHORT).show();
                break;
            case BiometricManager.BIOMETRIC_ERROR_NONE_ENROLLED:
                // Prompts the user to create credentials that your app accepts.
                final Intent enrollIntent = new Intent(Settings.ACTION_BIOMETRIC_ENROLL);
                enrollIntent.putExtra(Settings.EXTRA_BIOMETRIC_AUTHENTICATORS_ALLOWED,
                        BIOMETRIC_STRONG | DEVICE_CREDENTIAL);
                startActivityForResult(enrollIntent, REQUEST_CODE);
                break;
        }//Switch

        //Codigo de la activi_login de la pagina oficial
        //Se ejecuta despues de hacer la lectura de la huella o rostro.
        executor = ContextCompat.getMainExecutor(this);
        biometricPrompt = new BiometricPrompt(MainActivity.this,
                executor, new BiometricPrompt.AuthenticationCallback() {
            @Override
            public void onAuthenticationError(int errorCode,
                                              @NonNull CharSequence errString) {
                super.onAuthenticationError(errorCode, errString);
                Toast.makeText(getApplicationContext(),
                                "Error en la Autenticación: " + errString, Toast.LENGTH_SHORT)
                        .show();
            }

            @Override
            public void onAuthenticationSucceeded(
                    @NonNull BiometricPrompt.AuthenticationResult result) {
                super.onAuthenticationSucceeded(result);
                //Indicar a que pantalla va a navegar en caso de que sea una Autenticación exitosa!
                startActivity(new Intent(MainActivity.this, Home.class));

                Toast.makeText(getApplicationContext(),
                        "Autenticación Exitosa!", Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onAuthenticationFailed() {
                super.onAuthenticationFailed();
                Toast.makeText(getApplicationContext(), "Autenticación falló",
                                Toast.LENGTH_SHORT)
                        .show();
            }
        });

        promptInfo = new BiometricPrompt.PromptInfo.Builder()
                .setTitle("Inicio de sesion en Bancoppel")
                .setSubtitle("Para acceder favor de colocar tu huella en el sensor o tu rostro frente a la camara")
                .setNegativeButtonText("O utiliza tu contraseña")
                .build();


        //Al tocar la imagen, realiza la lectura de la biometria
        imagenLogin.setOnClickListener(view -> {
            biometricPrompt.authenticate(promptInfo);
        });

        //Al presionar el boton, realiza la lectura de la biometria
        botonLogin.setOnClickListener(view -> {
            biometricPrompt.authenticate(promptInfo);
        });

    }//onCreate
}//Class
