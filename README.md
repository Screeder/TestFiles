private static void Drawing_OnEndScene(EventArgs args)
{
	if (Device == null || Device.IsDisposed)
	{
		return;
	}

	Device.SetRenderState(RenderState.AlphaBlendEnable, true);

	foreach (var renderObject in _renderVisibleObjects)
	{
		if (renderObject is Sprite)
		{
			SpriteManager.Add(renderObject);
		}
		else
		{
			renderObject.OnEndScene();
		}
	}
	SpriteManager.OnEndScene();
}

public class TextManager
{
	private static List<Text> _texts = new List<Text>();
	private static readonly SharpDX.Direct3D9.Sprite _sprite = new SharpDX.Direct3D9.Sprite(Device);
	
	public void Add(Text text)
	{
		_texts.Add(text);
	}
	
	private void Clear()
	{
		_texts.RemoveAll();
	}
	
	public void OnEndScene()
	{
		try
		{
			if (_sprite.IsDisposed)
			{
				return;
			}
			
			_sprite.Begin(SpriteFlags.SortTexture | SpriteFlags.SortDepthBackToFront | SpriteFlags.DoNotAddRefTexture);
			var matrix = _sprite.Transform;
			foreach (Text text in _texts)
			{
				if (text.IsDisposed() || text.text == "")
				{
					return;
				}

				if (text.Unit != null && text.Unit.IsValid)
				{
					var pos = text.Unit.HPBarPosition + Offset;
					X = (int) pos.X;
					Y = (int) pos.Y;
				}

				var xP = text.X;
				var yP = text.Y;
				
				var nMatrix = Matrix.Translation(xP, yP, sprite.Layer);
				_sprite.Transform = nMatrix;
				if (text.OutLined)
				{
					var outlineColor = new ColorBGRA(0, 0, 0, 255);
					_textFont.DrawText(_sprite, text.text, xP - 1, yP - 1, outlineColor);
					_textFont.DrawText(_sprite, text.text, xP + 1, yP + 1, outlineColor);
					_textFont.DrawText(_sprite, text.text, xP - 1, yP, outlineColor);
					_textFont.DrawText(_sprite, text.text, xP + 1, yP, outlineColor);
				}
				_textFont.DrawText(_sprite, text.text, xP, yP, Color);
			}
			_sprite.Transform = matrix;
			_sprite.End();
			Clear();
		}
		catch (Exception e)
		{
			Reset();
			Console.WriteLine(@"Common.Render.TextManager.OnEndScene: " + e);
		}
	}
}

public class Text : RenderObject
{
	public delegate Vector2 PositionDelegate();

	public delegate string TextDelegate();

	private string _text;
	private Font _textFont;
	private int _x;
	private int _y;
	public bool Centered = false;
	public Vector2 Offset;
	public bool OutLined = false;
	public PositionDelegate PositionUpdate;
	public TextDelegate TextUpdate;
	public Obj_AI_Base Unit;

	public Text(string text, int x, int y, int size, ColorBGRA color, string fontName = "Calibri")
	{
		Color = color;
		this.text = text;

		_x = x;
		_y = y;

		_textFont = new Font(
			Device,
			new FontDescription
			{
				FaceName = fontName,
				Height = size,
				OutputPrecision = FontPrecision.Default,
				Quality = FontQuality.Default
			});
	}

	public Text(string text, Vector2 position, int size, ColorBGRA color, string fontName = "Calibri")
	{
		Color = color;
		this.text = text;

		_x = (int) position.X;
		_y = (int) position.Y;

		_textFont = new Font(
			Device,
			new FontDescription
			{
				FaceName = fontName,
				Height = size,
				OutputPrecision = FontPrecision.Default,
				Quality = FontQuality.Default
			});
	}

	public Text(string text,
		Obj_AI_Base unit,
		Vector2 offset,
		int size,
		ColorBGRA color,
		string fontName = "Calibri")
	{
		Unit = unit;
		Color = color;
		this.text = text;
		Offset = offset;

		var pos = unit.HPBarPosition + offset;

		_x = (int) pos.X;
		_y = (int) pos.Y;

		_textFont = new Font(
			Device,
			new FontDescription
			{
				FaceName = fontName,
				Height = size,
				OutputPrecision = FontPrecision.Default,
				Quality = FontQuality.Default
			});
	}

	public Text(int x, int y, string text, int size, ColorBGRA color, string fontName = "Calibri")
	{
		Color = color;
		this.text = text;

		_x = x;
		_y = y;

		_textFont = new Font(
			Device,
			new FontDescription
			{
				FaceName = fontName,
				Height = size,
				OutputPrecision = FontPrecision.Default,
				Quality = FontQuality.Default
			});
	}

	public Text(Vector2 position, string text, int size, ColorBGRA color, string fontName = "Calibri")
	{
		Color = color;
		this.text = text;
		_x = (int) position.X;
		_y = (int) position.Y;
		_textFont = new Font(
			Device,
			new FontDescription
			{
				FaceName = fontName,
				Height = size,
				OutputPrecision = FontPrecision.Default,
				Quality = FontQuality.Default
			});
	}

	public FontDescription TextFontDescription
	{
		get { return _textFont.Description; }

		set
		{
			_textFont.Dispose();
			_textFont = new Font(Device, value);
		}
	}

	public int X
	{
		get
		{
			var dx = Centered ? -_textFont.MeasureText(null, text, FontDrawFlags.Center).Width / 2 : 0;

			if (PositionUpdate != null)
			{
				return (int) PositionUpdate().X + dx;
			}

			return _x + dx;
		}
		set { _x = value; }
	}

	public int Y
	{
		get
		{
			var dy = Centered ? -_textFont.MeasureText(null, text, FontDrawFlags.Center).Height / 2 : 0;

			if (PositionUpdate != null)
			{
				return (int) PositionUpdate().Y + dy;
			}
			return _y + dy;
		}
		set { _y = value; }
	}

	public int Width { get; set; }
	public ColorBGRA Color { get; set; }

	public string text
	{
		get
		{
			if (TextUpdate != null)
			{
				return TextUpdate();
			}
			return _text;
		}
		set { _text = value; }
	}

	public override void OnEndScene()
	{
		try
		{
			if (_textFont.IsDisposed || text == "")
			{
				return;
			}

			if (Unit != null && Unit.IsValid)
			{
				var pos = Unit.HPBarPosition + Offset;
				X = (int) pos.X;
				Y = (int) pos.Y;
			}

			var xP = X;
			var yP = Y;
			if (OutLined)
			{
				var outlineColor = new ColorBGRA(0, 0, 0, 255);
				_textFont.DrawText(null, text, xP - 1, yP - 1, outlineColor);
				_textFont.DrawText(null, text, xP + 1, yP + 1, outlineColor);
				_textFont.DrawText(null, text, xP - 1, yP, outlineColor);
				_textFont.DrawText(null, text, xP + 1, yP, outlineColor);
			}
			_textFont.DrawText(null, text, xP, yP, Color);
		}
		catch (Exception e)
		{
			Console.WriteLine(@"Common.Render.Text.OnEndScene: " + e);
		}
	}

	public override void OnPreReset()
	{
		_textFont.OnLostDevice();
	}

	public override void OnPostReset()
	{
		_textFont.OnResetDevice();
	}

	public override void Dispose()
	{
		if (!_textFont.IsDisposed)
		{
			_textFont.Dispose();
		}
	}
}

public class SpriteManager
{
	private static List<Sprite> _sprites = new List<Sprite>();
	private static readonly SharpDX.Direct3D9.Sprite _sprite = new SharpDX.Direct3D9.Sprite(Device);
	
	public void Add(Sprite sprite)
	{
		_sprites.Add(sprite);
	}
	
	private void Clear()
	{
		_sprites.RemoveAll();
	}
	
	public void OnEndScene()
	{
		try
		{
			if (_sprite.IsDisposed)
			{
				return;
			}
			
			_sprite.Begin(SpriteFlags.SortTexture | SpriteFlags.SortDepthBackToFront | SpriteFlags.DoNotAddRefTexture);
			var matrix = _sprite.Transform;
			foreach (Sprite sprite in _sprites)
			{
				if (sprite._texture.IsDisposed || !sprite.Position.IsValid() || sprite.VisibleState())
				{
					return;
				}

				var nMatrix = (Matrix.Scaling(Scale.X, Scale.Y, 0)) * Matrix.RotationZ(Rotation) *
							  Matrix.Translation(Position.X, Position.Y, sprite.Layer);
				_sprite.Transform = nMatrix;
				_sprite.Draw(_texture, _color, _crop);
			}
			_sprite.Transform = matrix;
			_sprite.End();
			Clear();
		}
		catch (Exception e)
		{
			Reset();
			Console.WriteLine(@"Common.Render.SpriteManager.OnEndScene: " + e);
		}
	}
}

public class Sprite : RenderObject
        {
            public delegate void OnResetting(Sprite sprite);

            public delegate Vector2 PositionDelegate();

            private readonly SharpDX.Direct3D9.Sprite _sprite = new SharpDX.Direct3D9.Sprite(Device);
            private ColorBGRA _color = SharpDX.Color.White;
            private SharpDX.Rectangle? _crop;
            private bool _hide;
            private Texture _originalTexture;
            private Vector2 _scale = new Vector2(1, 1);
            private Texture _texture;
            private int _x;
            private int _y;

            public Sprite(Bitmap bitmap, Vector2 position)
            {
                UpdateTextureBitmap(bitmap, position);
            }

            public Sprite(BaseTexture texture, Vector2 position)
            {
                UpdateTextureBitmap(
                    (Bitmap) Image.FromStream(BaseTexture.ToStream(texture, ImageFileFormat.Bmp)), position);
            }

            public Sprite(Stream stream, Vector2 position)
            {
                UpdateTextureBitmap(new Bitmap(stream), position);
            }

            public Sprite(byte[] bytesArray, Vector2 position)
            {
                UpdateTextureBitmap((Bitmap) Image.FromStream(new MemoryStream(bytesArray)), position);
            }

            public Sprite(string fileLocation, Vector2 position)
            {
                if (!File.Exists((fileLocation)))
                {
                    return;
                }

                UpdateTextureBitmap(new Bitmap(fileLocation), position);
            }

            public int X
            {
                get
                {
                    if (PositionUpdate != null)
                    {
                        return (int) PositionUpdate().X;
                    }
                    return _x;
                }
                set { _x = value; }
            }

            public int Y
            {
                get
                {
                    if (PositionUpdate != null)
                    {
                        return (int) PositionUpdate().Y;
                    }
                    return _y;
                }
                set { _y = value; }
            }

            public Bitmap Bitmap { get; set; }

            public int Width
            {
                get { return (int) (Bitmap.Width * _scale.X); }
            }

            public int Height
            {
                get { return (int) (Bitmap.Height * _scale.Y); }
            }

            public Vector2 Size
            {
                get { return new Vector2(Bitmap.Width, Bitmap.Height); }
            }

            public Vector2 Position
            {
                set
                {
                    X = (int) value.X;
                    Y = (int) value.Y;
                }

                get { return new Vector2(X, Y); }
            }

            public PositionDelegate PositionUpdate { get; set; }

            public Vector2 Scale
            {
                set { _scale = value; }
                get { return _scale; }
            }

            public float Rotation { set; get; }

            public ColorBGRA Color
            {
                set { _color = value; }
                get { return _color; }
            }

            public event OnResetting OnReset;

            public void Crop(int x, int y, int w, int h, bool scale = false)
            {
                _crop = new SharpDX.Rectangle(x, y, w, h);

                if (scale)
                {
                    _crop = new SharpDX.Rectangle(
                        (int) (_scale.X * x), (int) (_scale.Y * y), (int) (_scale.X * w), (int) (_scale.Y * h));
                }
            }

            public void Crop(SharpDX.Rectangle rect, bool scale = false)
            {
                _crop = rect;

                if (scale)
                {
                    _crop = new SharpDX.Rectangle(
                        (int) (_scale.X * rect.X), (int) (_scale.Y * rect.Y), (int) (_scale.X * rect.Width),
                        (int) (_scale.Y * rect.Height));
                }
            }

            public void Show()
            {
                _hide = false;
            }

            public void Hide()
            {
                _hide = true;
            }
			
			public bool VisibleState()
            {
                return _hide;
            }

            public void Reset()
            {
                UpdateTextureBitmap(
                    (Bitmap) Image.FromStream(BaseTexture.ToStream(_originalTexture, ImageFileFormat.Bmp)));

                if (OnReset != null)
                {
                    OnReset(this);
                }
            }

            public void GrayScale()
            {
                SetSaturation(0.0f);
            }

            public void Fade()
            {
                SetSaturation(0.5f);
            }

            public void Complement()
            {
                SetSaturation(-1.0f);
            }

            public void SetSaturation(float saturiation)
            {
                UpdateTextureBitmap(SaturateBitmap(Bitmap, saturiation));
            }

            private static Bitmap SaturateBitmap(Image original, float saturation)
            {
                const float rWeight = 0.3086f;
                const float gWeight = 0.6094f;
                const float bWeight = 0.0820f;

                var a = (1.0f - saturation) * rWeight + saturation;
                var b = (1.0f - saturation) * rWeight;
                var c = (1.0f - saturation) * rWeight;
                var d = (1.0f - saturation) * gWeight;
                var e = (1.0f - saturation) * gWeight + saturation;
                var f = (1.0f - saturation) * gWeight;
                var g = (1.0f - saturation) * bWeight;
                var h = (1.0f - saturation) * bWeight;
                var i = (1.0f - saturation) * bWeight + saturation;

                var newBitmap = new Bitmap(original.Width, original.Height);
                var gr = Graphics.FromImage(newBitmap);

                // ColorMatrix elements
                float[][] ptsArray =
                {
                    new[] { a, b, c, 0, 0 }, new[] { d, e, f, 0, 0 }, new[] { g, h, i, 0, 0 },
                    new float[] { 0, 0, 0, 1, 0 }, new float[] { 0, 0, 0, 0, 1 }
                };
                // Create ColorMatrix
                var clrMatrix = new ColorMatrix(ptsArray);
                // Create ImageAttributes
                var imgAttribs = new ImageAttributes();
                // Set color matrix
                imgAttribs.SetColorMatrix(clrMatrix, ColorMatrixFlag.Default, ColorAdjustType.Default);
                // Draw Image with no effects
                gr.DrawImage(original, 0, 0, original.Width, original.Height);
                // Draw Image with image attributes
                gr.DrawImage(
                    original, new System.Drawing.Rectangle(0, 0, original.Width, original.Height), 0, 0, original.Width,
                    original.Height, GraphicsUnit.Pixel, imgAttribs);
                gr.Dispose();

                return newBitmap;
            }

            public void UpdateTextureBitmap(Bitmap newBitmap, Vector2 position = new Vector2())
            {
                if (position.IsValid())
                {
                    Position = position;
                }

                if (Bitmap != null)
                {
                    Bitmap.Dispose();
                }
                Bitmap = newBitmap;

                _texture = Texture.FromMemory(
                    Device, (byte[]) new ImageConverter().ConvertTo(newBitmap, typeof(byte[])), Width, Height, 0,
                    Usage.None, Format.A1, Pool.Managed, Filter.Default, Filter.Default, 0);

                if (_originalTexture == null)
                {
                    _originalTexture = _texture;
                }
            }

            public override void OnEndScene()
            {
            }

            public override void OnPreReset()
            {
            }

            public override void OnPostReset()
            {
            }

            public override void Dispose()
            {
                if (!_texture.IsDisposed)
                {
                    _texture.Dispose();
                }
            }
			
			public bool IsDisposed()
			{
				_texture.IsDisposed;
			}
        }
