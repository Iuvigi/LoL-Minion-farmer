using System;
using System.Collections.Generic;
using System.Linq;
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Audio;
using Microsoft.Xna.Framework.Content;
using Microsoft.Xna.Framework.GamerServices;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;
using Microsoft.Xna.Framework.Media;
using LoLMinionFarmer;

namespace LoLMinionFarmer
{
    /// <summary>
    /// This is the main type for my game *created with the help of Dr. Tim's teddybears
    /// </summary>
    public class Game1 : Microsoft.Xna.Framework.Game
    {
        GraphicsDeviceManager graphics;
        SpriteBatch spriteBatch;

        // click support
        ButtonState previousButtonState = ButtonState.Released;

        //shooting support
        Arrow arrow;
        private Minion theCurrentTarget;

        //mousing support
        MouseState mouse;

        //champion support
        Champion vayne;

        //screen support
        public const int WINDOW_WIDTH = 800;
        public const int WINDOW_HEIGHT = 600;
        const int INITIAL_NUM_MINIONS = 1;


        // random support & lists of minions
        Random rand = new Random();
        List<Texture2D> friendlySprites = new List<Texture2D>();
        List<Texture2D> enemySprites = new List<Texture2D>();

        // spawning support
        const int TOTAL_enemy_SPAWN_DELAY_MILLISECONDS = 400;
        const int TOTAL_friendly_SPAWN_DELAY_MILLISECONDS = 400;
        int elapsedFriendlySpawnDelayMilliseconds = 0;
        int elapsedEnemySpawnDelayMilliseconds = 0;
        int numberOfFriendlyMinionsToSpawn = 0;
        int numberOfEnemyMinionsToSpawn = 0;

        // game objects
        List<Minion> enemyMinions = new List<Minion>();
        List<Minion> friendlyMinions = new List<Minion>();
        

        public Game1()
        {
            graphics = new GraphicsDeviceManager(this);
            Content.RootDirectory = "Content";

            graphics.PreferredBackBufferWidth = WINDOW_WIDTH;
            graphics.PreferredBackBufferHeight = WINDOW_HEIGHT;

            IsMouseVisible = true;
        }

        /// <summary>
        /// Allows the game to perform any initialization it needs to before starting to run.
        /// This is where it can query for any required services and load any non-graphic
        /// related content.  Calling base.Initialize will enumerate through any components
        /// and initialize them as well.
        /// </summary>
        protected override void Initialize()
        {
            // TODO: Add your initialization logic here

            base.Initialize();
        }

        /// <summary>
        /// LoadContent will be called once per game and is the place to load
        /// all of your content.
        /// </summary>
        protected override void LoadContent()
        {
            // Create a new SpriteBatch, which can be used to draw textures.
            spriteBatch = new SpriteBatch(GraphicsDevice);

            // load sprites for efficiency
            friendlySprites.Add(Content.Load<Texture2D>("smallMinion"));
            friendlySprites.Add(Content.Load<Texture2D>("bigMinion"));
            friendlySprites.Add(Content.Load<Texture2D>("smallMinion1"));
            enemySprites.Add(Content.Load<Texture2D>("smallMinionOF"));
            enemySprites.Add(Content.Load<Texture2D>("bigMinionOF"));
            enemySprites.Add(Content.Load<Texture2D>("smallMinion1OF"));

            //create Vayne Champion
            vayne = new Champion(Content.Load<Texture2D>("vayne"), new Vector2(WINDOW_WIDTH / 2, WINDOW_HEIGHT / 2), 4, 1 / 2, 50, 300, true);

            //create arrow(bullet) within the Champion
            arrow = new Arrow(Content.Load<Texture2D>("bullet"), new Vector2((int)(vayne.X), (int)(vayne.Y)),
                                             new Vector2((int)(vayne.X), (int)(vayne.Y)), 20, 10, 50, false);


            //create initial game objects - enemy minions
            for (int i = 0; i < INITIAL_NUM_MINIONS; i++)
            {
                enemyMinions.Add(GetEnemyMinion());
            }

            //create initial game objects - friendly minions
            for (int i = 0; i < INITIAL_NUM_MINIONS; i++)
            {
                friendlyMinions.Add(GetFriendlyMinion());
            }
        }

        /// <summary>
        /// UnloadContent will be called once per game and is the place to unload
        /// all content.
        /// </summary>
        protected override void UnloadContent()
        {
            // TODO: Unload any non ContentManager content here
        }

        /// <summary>
        /// Allows the game to run logic such as updating the world,
        /// checking for collisions, gathering input, and playing audio.
        /// </summary>
        /// <param name="gameTime">Provides a snapshot of timing values.</param>
        protected override void Update(GameTime gameTime)
        {
            // Allows the game to exit
            if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed)
                this.Exit();

            // update game entities
            MouseState mouse = Mouse.GetState();

            //checks if arrow intersects with a minion and applies damage to the minion
            for (int i = 0; i < enemyMinions.Count; i++)
            {
                if (enemyMinions[i].Active &&
                    enemyMinions[i].CollisionRectangle.Intersects(arrow.CollisionRectangle) && 
                    arrow.IsArrowActive)
                {
                    enemyMinions[i].TakeDamage(arrow.DamageDone);
                }
            }


            //checks if rightmouse button was clicked to activate the arrow
            if (mouse.RightButton == ButtonState.Released && previousButtonState == ButtonState.Pressed)
            {
                //checks if the right mouse click was on a minion and the minion is in champion's range
                for (int i = 0; i < enemyMinions.Count; i++)
                {
                    if (enemyMinions[i].CollisionRectangle.Contains(mouse.X, mouse.Y)
                         && Math.Sqrt((vayne.X - enemyMinions[i].X) * (vayne.X - enemyMinions[i].X) + 
                         (vayne.Y - enemyMinions[i].Y) * (vayne.Y - enemyMinions[i].Y)) < vayne.ChampionRange)
                    {
                        //mark the minion that the click was made on
                        theCurrentTarget = enemyMinions[i];

                        //sets the arrow.targetEntity to that particular minion
                        arrow.TargetX = theCurrentTarget.X;
                        arrow.TargetY = theCurrentTarget.Y;

                        arrow.IsArrowActive = true;
                    }
                }
            }
            previousButtonState = mouse.RightButton;


            // spawn enemy minions as appropriate
            elapsedEnemySpawnDelayMilliseconds += gameTime.ElapsedGameTime.Milliseconds;
            if (elapsedEnemySpawnDelayMilliseconds > TOTAL_enemy_SPAWN_DELAY_MILLISECONDS && numberOfEnemyMinionsToSpawn < 5)
                {
                elapsedEnemySpawnDelayMilliseconds = 0;
                enemyMinions.Add(GetEnemyMinion());
                numberOfEnemyMinionsToSpawn++;
                }

            // spawn friendly minions as appropriate
            elapsedFriendlySpawnDelayMilliseconds += gameTime.ElapsedGameTime.Milliseconds;
            if (elapsedFriendlySpawnDelayMilliseconds > TOTAL_friendly_SPAWN_DELAY_MILLISECONDS && numberOfFriendlyMinionsToSpawn < 5)
            {
                elapsedFriendlySpawnDelayMilliseconds = 0;
                friendlyMinions.Add(GetFriendlyMinion());
                numberOfFriendlyMinionsToSpawn++;
            }

            // update enemy minions
            for (int i = 0; i < enemyMinions.Count; i++)
            {
                enemyMinions[i].Update(gameTime);
            }

            // update friendly minions
            for (int i = 0; i < friendlyMinions.Count; i++)
            {
                friendlyMinions[i].Update(gameTime);
            }

            //update champion
            vayne.Update(gameTime, mouse);

            //update arrow (bullet)
            arrow.Update(gameTime, mouse);

            base.Update(gameTime);
        }

        /// <summary>
        /// This is called when the game should draw itself.
        /// </summary>
        /// <param name="gameTime">Provides a snapshot of timing values.</param>
        protected override void Draw(GameTime gameTime)
        {
            GraphicsDevice.Clear(Color.CornflowerBlue);

            spriteBatch.Begin();

            //draw friendly minions
            for (int i = 0; i < friendlyMinions.Count; i++)
            {
                friendlyMinions[i].Draw(spriteBatch);
            }

            //draw enemy minions
            for (int i = 0; i < enemyMinions.Count; i++)
            {
                enemyMinions[i].Draw(spriteBatch);
            }

            //draw champion
            vayne.Draw(spriteBatch);

            //draw arrow
            if (arrow.IsArrowActive)
            {
                arrow.Draw(spriteBatch);
            }
            
            
            spriteBatch.End();

            base.Draw(gameTime);
        }

        /// <summary>
        /// Gets an enemy minion with HP=200
        /// </summary>
        /// <returns>enemy minion</returns>
        private Minion GetEnemyMinion()
        {
            Texture2D sprite = enemySprites[rand.Next(2)];
            return new Minion(sprite, WINDOW_WIDTH, 0, WINDOW_WIDTH, WINDOW_HEIGHT, false, 200);
        }

        /// <summary>
        /// Gets a friendly minion with HP=200
        /// </summary>
        /// <returns>friendly minion</returns>
        private Minion GetFriendlyMinion()
        {
            Texture2D sprite = friendlySprites[rand.Next(2)];
            return new Minion(sprite, 0, WINDOW_HEIGHT, WINDOW_WIDTH, WINDOW_HEIGHT, true, 200);
        }
    }
}

