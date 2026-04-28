const {
  Client,
  GatewayIntentBits,
  PermissionsBitField,
  ChannelType,
  ActionRowBuilder,
  ButtonBuilder,
  ButtonStyle,
  EmbedBuilder,
  Events,
  REST,
  Routes,
  SlashCommandBuilder
} = require('discord.js');

const TOKEN = 'MTQ5ODc1NzY2ODExMjk1NzUxMg.G1WMa9.o2pIpBOebbMEmVXvkETibSrPFI0vqDwhSizoMw';
const CLIENT_ID = '1498757668112957512';

const client = new Client({
  intents: [GatewayIntentBits.Guilds]
});

// ================= REGISTER COMMANDS =================
client.once(Events.ClientReady, async () => {
  console.log(`Logged in as ${client.user.tag}`);

  const commands = [
    new SlashCommandBuilder()
      .setName('panel')
      .setDescription('Send ticket panel'),

    new SlashCommandBuilder()
      .setName('close')
      .setDescription('Close a ticket')
  ].map(c => c.toJSON());

  const rest = new REST({ version: '10' }).setToken(TOKEN);

  await rest.put(
    Routes.applicationCommands(CLIENT_ID),
    { body: commands }
  );

  console.log('Commands loaded');
});

// ================= INTERACTIONS =================
client.on(Events.InteractionCreate, async interaction => {

  try {

    // ================= SLASH COMMANDS =================
    if (interaction.isChatInputCommand()) {

      // PANEL COMMAND
      if (interaction.commandName === 'panel') {

        const embed = new EmbedBuilder()
          .setTitle('🎫 Support Ticket System')
          .setDescription(`
Click a button below:

🛒 Buy Support
❓ Help Support
🚨 Report User
          `)
          .setColor(0x00AEFF)
          .setImage('YOUR_IMAGE_LINK_HERE');

        const row = new ActionRowBuilder().addComponents(
          new ButtonBuilder()
            .setCustomId('ticket_buy')
            .setLabel('Buy')
            .setStyle(ButtonStyle.Primary),

          new ButtonBuilder()
            .setCustomId('ticket_help')
            .setLabel('Help')
            .setStyle(ButtonStyle.Success),

          new ButtonBuilder()
            .setCustomId('ticket_report')
            .setLabel('Report')
            .setStyle(ButtonStyle.Danger)
        );

        return interaction.reply({
          embeds: [embed],
          components: [row]
        });
      }

      // CLOSE COMMAND
      if (interaction.commandName === 'close') {

        if (!interaction.channel.name.includes('ticket')) {
          return interaction.reply({
            content: '❌ This is not a ticket channel.',
            ephemeral: true
          });
        }

        await interaction.reply({
          content: '🔒 Closing ticket...',
          ephemeral: true
        });

        setTimeout(() => {
          interaction.channel.delete().catch(() => {});
        }, 3000);
      }
    }

    // ================= BUTTONS =================
    if (interaction.isButton()) {

      await interaction.deferReply({ ephemeral: true });

      let type = null;

      if (interaction.customId === 'ticket_buy') type = 'buy';
      if (interaction.customId === 'ticket_help') type = 'help';
      if (interaction.customId === 'ticket_report') type = 'report';
      if (interaction.customId === 'ticket_close') type = 'close';

      // ================= CREATE TICKET =================
      if (type && type !== 'close') {

        const channel = await interaction.guild.channels.create({
          name: `${type}-ticket-${interaction.user.username}`,
          type: ChannelType.GuildText,

          permissionOverwrites: [
            {
              id: interaction.guild.id,
              deny: [PermissionsBitField.Flags.ViewChannel]
            },
            {
              id: interaction.user.id,
              allow: [
                PermissionsBitField.Flags.ViewChannel,
                PermissionsBitField.Flags.SendMessages,
                PermissionsBitField.Flags.ReadMessageHistory
              ]
            }
          ]
        });

        const closeBtn = new ActionRowBuilder().addComponents(
          new ButtonBuilder()
            .setCustomId('ticket_close')
            .setLabel('🔒 Close Ticket')
            .setStyle(ButtonStyle.Danger)
        );

        await channel.send({
          content: `👋 Welcome ${interaction.user}`,
          components: [closeBtn]
        });

        return interaction.editReply({
          content: `✅ Ticket created: ${channel}`
        });
      }

      // ================= CLOSE BUTTON =================
      if (type === 'close') {

        await interaction.editReply({
          content: '🔒 Closing ticket...'
        });

        setTimeout(() => {
          interaction.channel.delete().catch(() => {});
        }, 3000);
      }
    }

  } catch (err) {
    console.log('ERROR:', err);

    if (!interaction.replied && !interaction.deferred) {
      await interaction.reply({
        content: '❌ Something went wrong.',
        ephemeral: true
      });
    }
  }
});

client.login(TOKEN);
