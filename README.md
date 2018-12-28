# Shark-Game
This is my first game, created with C# Visual Studio in programming course at University of Jyväskylä

using System;
using System.Collections.Generic;
using Jypeli;
using Jypeli.Assets;
using Jypeli.Controls;
using Jypeli.Widgets;
/// @author  Otto Schildt
/// @version 2.12.2018
/// <summary>
/// Peli, jossa valkohai yrittää kasvaa mahdollisimman isoksi
/// </summary>
public class HaiPeli : PhysicsGame
{
    private Image[] PelinKuvat = LoadImages("valkohai", "taustakuva", "sukeltaja", "muovi", "yacht");
    private IntMeter pisteLaskuri;
    private PhysicsObject hai;
    private const int scrollausnopeus = -5;
    private List<GameObject> taustakuvat;
    private Timer taustaAjastin = new Timer();
    private GameObject ekaTaustakuva;
    private Timer ajastin;
    /// <summary>
    /// Aloitetaan peli. 
    /// </summary>
    /// 
    public override void Begin()
    {
        LuoKentta();
        AsetaOhjaimet();
        LuoPistelaskuri();
    }


    /// <summary>
    /// Kun pelaaja on saanut 30 pistettä, peli pysähtyy.
    /// </summary>
    private void KaikkiSyoty()
    {
        MessageDisplay.Add("Selvisit!");
        IsPaused = true;
    }


    /// <summary>
    /// Luodaan liikkuva taustakuva
    /// </summary>
    private void LuoTaustakuvat()
    {
        taustaAjastin = new Timer();
        taustaAjastin.Interval = 0.01;   // tällä voit myös säätää nopeutta
        taustaAjastin.Timeout += LiikutaTaustaa;
        taustaAjastin.Start();
        taustakuvat = new List<GameObject>();
        for (int i = 2; i < 100; i++)
        {
            LisaaTaustakuva("taustakuva", 800, 800);
        }
        Console.WriteLine("GAME OVER");
    }


    /// <summary>
    /// Luodaan liikkuva taustakuva
    /// </summary>
    /// <param name="tausta >tausta</param>
    private void LisaaTaustakuva(string nimi, double leveys, double korkeus)
    {
        GameObject tausta = new GameObject(leveys, korkeus);
        tausta.Image = PelinKuvat[1];
        tausta.X = 0;
        Add(tausta, -1);
        if (taustakuvat.Count > 0)
        {
            tausta.Right = taustakuvat[taustakuvat.Count - 1].Left;
            if (scrollausnopeus >= 0) ekaTaustakuva = tausta;
        }
        else
        {
            tausta.Right = Level.Right;
            if (scrollausnopeus < 0) ekaTaustakuva = tausta;
        }
        taustakuvat.Add(tausta);
    }


    /// <summary>
    /// Laitetaaan taustakuva liikkumaan
    /// </summary>
    private void LiikutaTaustaa()
    {
        foreach (GameObject taustakuva in taustakuvat)
        {
            taustakuva.X += scrollausnopeus;

            if (scrollausnopeus < 0 && taustakuva.Right < Level.Left)
            {
                taustakuva.Left = ekaTaustakuva.Right;
                ekaTaustakuva = taustakuva;
            }
            else if (scrollausnopeus > 0 && taustakuva.Left > Level.Right)
            {
                taustakuva.Right = ekaTaustakuva.Left;
                ekaTaustakuva = taustakuva;
            }
        }
    }


    /// <summary>
    /// Luodaan  kenttä ja siihen peliobjektit
    /// </summary>
    /// <param name="tausta >tausta</param>
    private void LuoKentta()
    {
        LisaaTaso(-83, 1200, 1);
        Gravity = new Vector(0, -200); 
        Camera.ZoomToLevel();
        Level.CreateBottomBorder(); 
        Level.CreateTopBorder(); 
        Camera.Follow(hai); 
        LuoTaustakuvat(); 
        LuoHai(-300, -200);
        AddCollisionHandler(hai, "vihollinen", HaiTormasi);
        AddCollisionHandler(hai, "seina", TormaaTasoon);
        LuoAjastimetLuoObjektille(2.0, 50, 50, PelinKuvat[2], 600, -500, 800, -150, 1.0);
        LuoAjastimetLuoObjektille(20.0, 200, 200, PelinKuvat[4], 600, -120, 800, -120, 1.5);
        LuoAjastimetLuoObjektille(3.0, 50, 50, PelinKuvat[3], 600, -500, 800, -150, 0.5);
    }


    /// <summary>
    /// Luodaan näkymätön taso, joka estää peliobjektien pääsyn pois vedestä
    /// </summary>
    private void LisaaTaso(double paikka, double leveys, double korkeus)
    {
        PhysicsObject taso = PhysicsObject.CreateStaticObject(leveys, korkeus);
        taso.Y = paikka;
        taso.X = 0;
        taso.Width = leveys;
        taso.Height = korkeus;
        taso.Tag = "seina";
        Add(taso, -1);
    }


    /// <summary>
    /// Luodaan pistelaskuri, kun hai osuu muihin olioihin.
    /// </summary>
    private void LuoPistelaskuri()
    {
        pisteLaskuri = new IntMeter(0);
        IntMeter syodytOliot = new IntMeter(0);
        pisteLaskuri.MaxValue = 10;
        pisteLaskuri.UpperLimit += KaikkiSyoty;
        Label pisteNaytto = new Label();
        pisteNaytto.Title = "Pisteet";
        pisteNaytto.X = Screen.Left + 100;
        pisteNaytto.Y = Screen.Top - 100;
        pisteNaytto.TextColor = Color.Black;
        pisteNaytto.Color = Color.White;
        pisteNaytto.BindTo(pisteLaskuri);
        Add(pisteNaytto);
    }


    /// <summary>
    /// Asetetaan ohjainnäppäimet pelaajalle.
    /// </summary>
    private void AsetaOhjaimet()
    {
        Keyboard.Listen(Key.W, ButtonState.Down, AsetaNopeus, "hai ylös", hai, new Vector(50, 400));
        Keyboard.Listen(Key.W, ButtonState.Released, AsetaNopeus, "hai sivulle", hai, new Vector(50, 0));
        Keyboard.Listen(Key.S, ButtonState.Down, AsetaNopeus, "hai alas", hai, new Vector(50, -400));
        Keyboard.Listen(Key.S, ButtonState.Released, AsetaNopeus, "hai sivulle", hai, new Vector(50, 0));
        Keyboard.Listen(Key.A, ButtonState.Down, AsetaNopeus, "hai taakse", hai, new Vector(-200, 0));
        PhoneBackButton.Listen(ConfirmExit, "Lopeta peli");
        Keyboard.Listen(Key.Escape, ButtonState.Pressed, ConfirmExit, "Lopeta peli");
    }


    /// <summary>
    /// Hain nopeus
    /// </summary>
    /// <param name=" hai </param>
    private void AsetaNopeus(PhysicsObject hai, Vector nopeus)
    {
        hai.Velocity = nopeus;
    }


    /// <summary>
    /// Luodaan peliobjekti, joka liikkuu vedessä
    /// </summary>
    private PhysicsObject LuoObjekti(double leveys, double korkeus, Image PelinKuvat, double minX, double minY, double maxX, double maxY, double massa)
    {
        PhysicsObject objekti = new PhysicsObject(leveys, korkeus);
        objekti.Image = PelinKuvat;
        objekti.Angle = Angle.FromDegrees(0);
        objekti.Shape = Shape.FromImage(PelinKuvat);
        objekti.CollisionIgnoreGroup = 1;
        objekti.Tag = "vihollinen";
        objekti.LifetimeLeft = TimeSpan.FromSeconds(40.0);
        objekti.Position = RandomGen.NextVector(minX, minY, maxX, maxY);
        LiikutaOliota(objekti, -30, 0, -600, 30);
        objekti.IgnoresGravity = true;
        objekti.Mass = massa;
        Add(objekti, 1);
        return objekti;
    }


    /// <summary>
    /// Luodaan impulssi, joka liikuttaa objektia
    /// </summary>
    private void LiikutaOliota(PhysicsObject olio, double minX, double minY, double maxX, double maxY)
    {
        Vector impulssi = RandomGen.NextVector(minX, minY, maxX, maxY);
        olio.Hit(impulssi);
    }


    /// <summary>
    /// Luodaan hai
    /// </summary>
    /// <param name="hai">pelin hahmo</param>
    private PhysicsObject LuoHai(double x, double y)
    {
        hai = new PhysicsObject(50, 50);
        hai.X = x;
        hai.Y = y;
        hai.Image = PelinKuvat[0];
        hai.Angle = Angle.FromDegrees(13);
        hai.IgnoresExplosions = true;
        hai.IgnoresGravity = false;
        hai.CanRotate = false;
        hai.Mass = 1.0;
        hai.Restitution = 0;
        Add(hai, 1);
        return (hai);
    }


    /// <summary>
    /// peliobjektin törmäys vesirajaan
    /// </summary>
    /// <param name="tormaaja">pelin hahmo</param>
    /// <param name="kohde">objekti johon törmättiin</param>
    private void TormaaTasoon(PhysicsObject tormaaja, PhysicsObject kohde)
    {
        if (tormaaja.Top == kohde.Bottom)
        {
            Keyboard.Disable(Key.W);
        }
    }


    /// <summary>
    /// aliohjelma, joka kutsuu tormayyksen vaikutusta
    /// </summary>
    /// <param name="hai">pelin hahmo</param>
    /// <param name="vihollinen">vihu johon törmättiin</param>
    private void HaiTormasi(PhysicsObject hai, PhysicsObject vihollinen)
    {
        TormayksenVaikutus(hai, vihollinen, 10, 10);
    }


    /// <summary>
    /// Luodaan aliohjelma, joka luo 
    /// </summary>
    /// <param name="olio">pelin hahmo</param>
    /// <param name="olio2">vihu johon törmättiin</param>
    public void TormayksenVaikutus(PhysicsObject olio, PhysicsObject olio2, int leveys, int korkeus)
    {
        if (hai.Mass < olio2.Mass)
        {
            Explosion rajahdys = new Explosion(olio2.Width * 2);
            rajahdys.Position = olio2.Position;
            rajahdys.UseShockWave = true;
            rajahdys.Force = 500;
            this.Add(rajahdys);
            olio.Width += leveys;
            olio.Height += korkeus;
            pisteLaskuri.Value += 5;
            Remove(olio2);
        }
        if (hai.Mass > olio2.Mass)
        {
            olio.Width -= leveys;
            olio.Height -= korkeus;
            olio.Color = Color.Green;
            pisteLaskuri.Value -= 2;
            Remove(olio2);
        }
        if (hai.Mass == olio2.Mass)
        {
            olio.Width += leveys;
            olio.Height += korkeus;
            pisteLaskuri.Value += 1;
            for (int i = 0; i < 100; i++)
            {
                LuoVerta(2, 2, 1);
            }
            Remove(olio2);
        }
    }


    /// <summary>
    /// Luodaan verta hain syödessä sukeltajan
    /// </summary>
    /// <param name="veri">veri pallo</param>
    private PhysicsObject LuoVerta(double x, double y, double sade)
    {
        PhysicsObject veri = new PhysicsObject(sade * 2, sade * 2, Shape.Circle);
        veri.X = hai.X + 10;
        veri.Y = hai.Y;
        veri.CollisionIgnoreGroup = 1;
        veri.Restitution = 0.0;
        veri.IgnoresGravity = true;
        Vector VeriImpulssi = RandomGen.NextVector(-40, -40, 40, 40);
        veri.Hit(VeriImpulssi);
        veri.LifetimeLeft = TimeSpan.FromSeconds(2.0);
        veri.Color = Color.BloodRed;
        Add(veri, 1);
        return veri;
    }


    /// <summary>
    /// Luodaan ajasimet LuoObjekti aliohjelmalle
    /// </summary>
    void LuoAjastimetLuoObjektille(double intervalli, double leveys, double korkeus, Image PelinKuvat, double minX, double minY, double maxX, double maxY, double massa)
    {
        Timer ajastin = new Timer(); //Luodaan laskuri
        ajastin.Interval = intervalli;   //Kerran (intervalli) sekunnissa toteuta LuoVene
        ajastin.Timeout += delegate
        {
            LuoObjekti(leveys, korkeus, PelinKuvat, minX, minY, maxX, maxY, massa);
        };
        ajastin.Start();
    }
}








