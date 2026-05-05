# Sentry Error Triage

When a new Sentry error fires, it pulls the trace, searches the web for similar bugs, and posts a starting hypothesis to the on-call channel. @mention it any time to debug live.

## Prerequisites
- A [Sentry](https://sentry.io) organization where you can create an internal integration (API access + webhook)
- A Sentry project configured to send issue webhooks to the agent's webhook URL after deploy
- A Slack workspace where you can install the agent's bot and invite it to one or more channels

<table>
  <tr>
    <td><strong>CHANNELS</strong></td>
    <td><code>slack</code> · <code>sentry-webhook</code></td>
  </tr>
  <tr>
    <td><strong>CONNECTORS</strong></td>
    <td><code>sentry-mcp</code> · <code>parallel-search-mcp</code></td>
  </tr>
  <tr>
    <td colspan="2" align="center">
      <br />
      <a href="https://valet.dev/deploy?from=github.com/valet-agents/sentry-error-triage">
        <img src="https://raw.githubusercontent.com/valet-agents/sentry-error-triage/main/.github/deploy-button.svg" alt="Deploy Agent →" height="40" />
      </a>
      <br /><br />
    </td>
  </tr>
</table>
