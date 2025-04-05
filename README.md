package com.example.aimdrag

import android.accessibilityservice.AccessibilityService import android.accessibilityservice.GestureDescription import android.graphics.Path import android.os.Handler import android.os.Looper import android.view.accessibility.AccessibilityEvent import android.widget.Toast import android.content.Intent import android.app.Service import android.app.PendingIntent import android.app.Notification import android.app.NotificationChannel import android.app.NotificationManager import android.os.Build import androidx.core.app.NotificationCompat

class AimDragService : AccessibilityService() {

private var isDragging = false
private val handler = Handler(Looper.getMainLooper())

private val dragRunnable = object : Runnable {
    override fun run() {
        if (isDragging) {
            performAimDrag()
            handler.postDelayed(this, 10)
        }
    }
}

override fun onServiceConnected() {
    super.onServiceConnected()
    startForegroundService()
}

override fun onAccessibilityEvent(event: AccessibilityEvent?) {}

override fun onInterrupt() {}

fun startDrag() {
    if (!isDragging) {
        isDragging = true
        handler.post(dragRunnable)
        Toast.makeText(this, "Aim Drag تشغيل", Toast.LENGTH_SHORT).show()
    }
}

fun stopDrag() {
    isDragging = false
    Toast.makeText(this, "Aim Drag إيقاف", Toast.LENGTH_SHORT).show()
}

private fun performAimDrag() {
    val path = Path().apply {
        moveTo(500f, 1500f)
        lineTo(500f, 1490f)
    }
    val gesture = GestureDescription.Builder()
        .addStroke(GestureDescription.StrokeDescription(path, 0, 10))
        .build()
    dispatchGesture(gesture, null, null)
}

private fun startForegroundService() {
    val channelId = "AimDragServiceChannel"
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        val channel = NotificationChannel(channelId, "Aim Drag Service", NotificationManager.IMPORTANCE_LOW)
        val manager = getSystemService(NotificationManager::class.java)
        manager.createNotificationChannel(channel)
    }

    val notification: Notification = NotificationCompat.Builder(this, channelId)
        .setContentTitle("Aim Drag يعمل")
        .setSmallIcon(android.R.drawable.ic_media_play)
        .build()

    startForeground(1, notification)
}

}

// MainActivity لإظهار زر التشغيل والإيقاف

package com.example.aimdrag

import android.content.Intent import android.os.Bundle import android.widget.Button import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity() {

private var isRunning = false
private lateinit var aimDragService: AimDragService

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    val startButton = findViewById<Button>(R.id.startButton)

    startButton.setOnClickListener {
        if (!isRunning) {
            val intent = Intent(this, AimDragService::class.java)
            startService(intent)
            isRunning = true
        } else {
            isRunning = false
            stopService(Intent(this, AimDragService::class.java))
        }
    }
}

}

// activity_main.xml

<?xml version="1.0" encoding="utf-8"?><LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
android:layout_width="match_parent"
android:layout_height="match_parent"
android:orientation="vertical"
android:gravity="center">

<Button
    android:id="@+id/startButton"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="تشغيل / إيقاف Aim Drag" />

</LinearLayout>
