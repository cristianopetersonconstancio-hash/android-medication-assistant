# Trechos Técnicos Relevantes

Este repositório contém **snippets isolados** de lógica crítica utilizados no desenvolvimento
de um aplicativo Android voltado à **segurança, acessibilidade e adesão correta a tratamentos médicos**.

Os códigos abaixo representam desafios reais enfrentados em versões recentes do Android
e as soluções adotadas.

---

## Acordar a tela e sobrepor o bloqueio (Android 10+)

```kotlin
private fun turnScreenOnAndKeyguard() {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O_MR1) {
        setShowWhenLocked(true)
        setTurnScreenOn(true)
        val keyguardManager =
            getSystemService(Context.KEYGUARD_SERVICE) as KeyguardManager
        keyguardManager.requestDismissKeyguard(this, null)
    } else {
        window.addFlags(
            WindowManager.LayoutParams.FLAG_SHOW_WHEN_LOCKED or
            WindowManager.LayoutParams.FLAG_DISMISS_KEYGUARD or
            WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON or
            WindowManager.LayoutParams.FLAG_TURN_SCREEN_ON
        )
    }
}
```

```kotlin
fun agendarAlarme(id: Int, horaMillis: Long) {

    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
        if (!alarmManager.canScheduleExactAlarms()) {
            solicitarPermissaoUsuario()
            return
        }
    }

    val intent = Intent(context, AlarmeReceiver::class.java).apply {
        putExtra("ID_REMEDIO", id)
    }

    val pendingIntent = PendingIntent.getBroadcast(
        context,
        id,
        intent,
        PendingIntent.FLAG_IMMUTABLE or PendingIntent.FLAG_UPDATE_CURRENT
    )

    alarmManager.setExactAndAllowWhileIdle(
        AlarmManager.RTC_WAKEUP,
        horaMillis,
        pendingIntent
    )
}
```

```kotlin
@Composable
fun CardMedicamento(medicamento: Medicamento, tempoAtual: Long) {

    val diferenca = medicamento.proximaDose - tempoAtual
    val ehHoraDeAgir = diferenca <= 0

    val corFundo =
        if (ehHoraDeAgir) Color.Red else Color(0xFF4CAF50)

    val podeClicar = ehHoraDeAgir

    Card(
        colors = CardDefaults.cardColors(containerColor = corFundo)
    ) {
        Row {
            Text(text = medicamento.nome, color = Color.White)

            IconButton(
                enabled = podeClicar,
                onClick = { marcarComoTomado(medicamento) }
            ) {
                if (podeClicar)
                    Icon(Icons.Default.CheckCircle, "Tomar")
                else
                    Icon(Icons.Default.Schedule, "Aguarde")
            }
        }
    }
}
```

```kotlin
override fun onReceive(context: Context, intent: Intent) {

    dispararAlarmeVisual(context, intent)

    val dezMinutosDepois =
        System.currentTimeMillis() + (10 * 60 * 1000)

    agendador.agendar(
        id = intent.getIntExtra("ID", 0),
        horaMillis = dezMinutosDepois
    )
}
```
