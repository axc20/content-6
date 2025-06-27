# Workers

```ruby
module MlAbstraction
  class SnapshotWorker < ApplicationWorker
    def perform
      bus_client = MlAbstraction::BusClient.new(BusContext.connection)
      snapshot = MlAbstraction::SnapshotService.call
      bus_client.publish_snapshot(snapshot)
    rescue StandardError => e
      Rails.logger.error "Error publishing snapshot: #{e.message}"
    end
  end
end
```

```yaml
# config/schedule.yml
ml_abstraction_snapshot:
  cron: '0 12 * * 1' # Every Monday at 12:00 (noon)
  class: 'MlAbstraction::SnapshotWorker'
  queue: default
```

---

```ruby
class InvitationTokenReminderWorker < ApplicationWorker
  def perform(id, remaining_expiration_time_in_hours = 24)
    invitation_token = InvitationToken.find(id)

    return unless invitation_token.is_eligible_for_client_sign_up_reminder?

    invitation_token.tokens_create_client.send!(remaining_expiration_time_in_hours)
  end
end

InvitationTokenReminderWorker.perform_in(24.hours, id)
```
