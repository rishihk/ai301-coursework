Addressed Qodo's first two findings: pause and resume now call `requireConnected` first (f13ee47), and the catalog budget comment no longer records change history (1a9d020).

On the third (the Paused badge): it's hidden on purpose while a rollout is held. In that state the status chip right next to it already reads "Rollout paused", so showing both would make the header say the same thing twice. When nothing is pending, the chip shows plain health, so the Paused badge appears and is the header's only paused signal. I'm happy to show both if you'd prefer.
