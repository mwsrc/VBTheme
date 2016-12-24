Imports System.Drawing.Drawing2D
Imports System.ComponentModel
Imports System.Runtime.InteropServices
'.::VibeLander Theme::.
'Author:   UnReLaTeDScript
'Credits:  Aeonhack [Themebase]
'Version:  1.0
MustInherit Class Theme
    Inherits ContainerControl

#Region " Initialization "

    Protected G As Graphics
    Sub New()
        SetStyle(DirectCast(139270, ControlStyles), True)
    End Sub

    Private ParentIsForm As Boolean
    Protected Overrides Sub OnHandleCreated(ByVal e As EventArgs)
        Dock = DockStyle.Fill
        ParentIsForm = TypeOf Parent Is Form
        If ParentIsForm Then
            If Not _TransparencyKey = Color.Empty Then ParentForm.TransparencyKey = _TransparencyKey
            ParentForm.FormBorderStyle = FormBorderStyle.None
        End If
        MyBase.OnHandleCreated(e)
    End Sub

    Overrides Property Text() As String
        Get
            Return MyBase.Text
        End Get
        Set(ByVal v As String)
            MyBase.Text = v
            Invalidate()
        End Set
    End Property
#End Region

#Region " Sizing and Movement "

    Private _Resizable As Boolean = True
    Property Resizable() As Boolean
        Get
            Return _Resizable
        End Get
        Set(ByVal value As Boolean)
            _Resizable = value
        End Set
    End Property

    Private _MoveHeight As Integer = 24
    Property MoveHeight() As Integer
        Get
            Return _MoveHeight
        End Get
        Set(ByVal v As Integer)
            _MoveHeight = v
            Header = New Rectangle(7, 7, Width - 14, _MoveHeight - 7)
        End Set
    End Property

    Private Flag As IntPtr
    Protected Overrides Sub OnMouseDown(ByVal e As MouseEventArgs)
        If Not e.Button = MouseButtons.Left Then Return
        If ParentIsForm Then If ParentForm.WindowState = FormWindowState.Maximized Then Return

        If Header.Contains(e.Location) Then
            Flag = New IntPtr(2)
        ElseIf Current.Position = 0 Or Not _Resizable Then
            Return
        Else
            Flag = New IntPtr(Current.Position)
        End If

        Capture = False
        DefWndProc(Message.Create(Parent.Handle, 161, Flag, Nothing))

        MyBase.OnMouseDown(e)
    End Sub

    Private Structure Pointer
        ReadOnly Cursor As Cursor, Position As Byte
        Sub New(ByVal c As Cursor, ByVal p As Byte)
            Cursor = c
            Position = p
        End Sub
    End Structure

    Private F1, F2, F3, F4 As Boolean, PTC As Point
    Private Function GetPointer() As Pointer
        PTC = PointToClient(MousePosition)
        F1 = PTC.X < 7
        F2 = PTC.X > Width - 7
        F3 = PTC.Y < 7
        F4 = PTC.Y > Height - 7

        If F1 And F3 Then Return New Pointer(Cursors.SizeNWSE, 13)
        If F1 And F4 Then Return New Pointer(Cursors.SizeNESW, 16)
        If F2 And F3 Then Return New Pointer(Cursors.SizeNESW, 14)
        If F2 And F4 Then Return New Pointer(Cursors.SizeNWSE, 17)
        If F1 Then Return New Pointer(Cursors.SizeWE, 10)
        If F2 Then Return New Pointer(Cursors.SizeWE, 11)
        If F3 Then Return New Pointer(Cursors.SizeNS, 12)
        If F4 Then Return New Pointer(Cursors.SizeNS, 15)
        Return New Pointer(Cursors.Default, 0)
    End Function

    Private Current, Pending As Pointer
    Private Sub SetCurrent()
        Pending = GetPointer()
        If Current.Position = Pending.Position Then Return
        Current = GetPointer()
        Cursor = Current.Cursor
    End Sub

    Protected Overrides Sub OnMouseMove(ByVal e As MouseEventArgs)
        If _Resizable Then SetCurrent()
        MyBase.OnMouseMove(e)
    End Sub

    Protected Header As Rectangle
    Protected Overrides Sub OnSizeChanged(ByVal e As EventArgs)
        If Width = 0 OrElse Height = 0 Then Return
        Header = New Rectangle(7, 7, Width - 14, _MoveHeight - 7)
        Invalidate()
        MyBase.OnSizeChanged(e)
    End Sub

#End Region

#Region " Convienence "

    MustOverride Sub PaintHook()
    Protected NotOverridable Overrides Sub OnPaint(ByVal e As PaintEventArgs)
        If Width = 0 OrElse Height = 0 Then Return
        G = e.Graphics
        PaintHook()
    End Sub

    Private _TransparencyKey As Color
    Property TransparencyKey() As Color
        Get
            Return _TransparencyKey
        End Get
        Set(ByVal v As Color)
            _TransparencyKey = v
            Invalidate()
        End Set
    End Property

    Private _Image As Image
    Property Image() As Image
        Get
            Return _Image
        End Get
        Set(ByVal value As Image)
            _Image = value
            Invalidate()
        End Set
    End Property
    ReadOnly Property ImageWidth() As Integer
        Get
            If _Image Is Nothing Then Return 0
            Return _Image.Width
        End Get
    End Property

    Private _Size As Size
    Private _Rectangle As Rectangle
    Private _Gradient As LinearGradientBrush
    Private _Brush As SolidBrush

    Protected Sub DrawCorners(ByVal c As Color, ByVal rect As Rectangle)
        _Brush = New SolidBrush(c)
        G.FillRectangle(_Brush, rect.X, rect.Y, 1, 1)
        G.FillRectangle(_Brush, rect.X + (rect.Width - 1), rect.Y, 1, 1)
        G.FillRectangle(_Brush, rect.X, rect.Y + (rect.Height - 1), 1, 1)
        G.FillRectangle(_Brush, rect.X + (rect.Width - 1), rect.Y + (rect.Height - 1), 1, 1)
    End Sub

    Protected Sub DrawBorders(ByVal p1 As Pen, ByVal p2 As Pen, ByVal rect As Rectangle)
        G.DrawRectangle(p1, rect.X, rect.Y, rect.Width - 1, rect.Height - 1)
        G.DrawRectangle(p2, rect.X + 1, rect.Y + 1, rect.Width - 3, rect.Height - 3)
    End Sub

    Protected Sub DrawText(ByVal a As HorizontalAlignment, ByVal c As Color, ByVal x As Integer)
        DrawText(a, c, x, 0)
    End Sub
    Protected Sub DrawText(ByVal a As HorizontalAlignment, ByVal c As Color, ByVal x As Integer, ByVal y As Integer)
        If String.IsNullOrEmpty(Text) Then Return
        _Size = G.MeasureString(Text, Font).ToSize
        _Brush = New SolidBrush(c)

        Select Case a
            Case HorizontalAlignment.Left
                G.DrawString(Text, Font, _Brush, x, _MoveHeight \ 2 - _Size.Height \ 2 + y)
            Case HorizontalAlignment.Right
                G.DrawString(Text, Font, _Brush, Width - _Size.Width - x, _MoveHeight \ 2 - _Size.Height \ 2 + y)
            Case HorizontalAlignment.Center
                G.DrawString(Text, Font, _Brush, Width \ 2 - _Size.Width \ 2 + x, _MoveHeight \ 2 - _Size.Height \ 2 + y)
        End Select
    End Sub

    Protected Sub DrawIcon(ByVal a As HorizontalAlignment, ByVal x As Integer)
        DrawIcon(a, x, 0)
    End Sub
    Protected Sub DrawIcon(ByVal a As HorizontalAlignment, ByVal x As Integer, ByVal y As Integer)
        If _Image Is Nothing Then Return
        Select Case a
            Case HorizontalAlignment.Left
                G.DrawImage(_Image, x, _MoveHeight \ 2 - _Image.Height \ 2 + y)
            Case HorizontalAlignment.Right
                G.DrawImage(_Image, Width - _Image.Width - x, _MoveHeight \ 2 - _Image.Height \ 2 + y)
            Case HorizontalAlignment.Center
                G.DrawImage(_Image, Width \ 2 - _Image.Width \ 2, _MoveHeight \ 2 - _Image.Height \ 2)
        End Select
    End Sub

    Protected Sub DrawGradient(ByVal c1 As Color, ByVal c2 As Color, ByVal x As Integer, ByVal y As Integer, ByVal width As Integer, ByVal height As Integer, ByVal angle As Single)
        _Rectangle = New Rectangle(x, y, width, height)
        _Gradient = New LinearGradientBrush(_Rectangle, c1, c2, angle)
        G.FillRectangle(_Gradient, _Rectangle)
    End Sub

#End Region

End Class
Module Draw
    Public Function RoundRect(ByVal Rectangle As Rectangle, ByVal Curve As Integer) As GraphicsPath
        Dim P As GraphicsPath = New GraphicsPath()
        Dim ArcRectangleWidth As Integer = Curve * 2
        P.AddArc(New Rectangle(Rectangle.X, Rectangle.Y, ArcRectangleWidth, ArcRectangleWidth), -180, 90)
        P.AddArc(New Rectangle(Rectangle.Width - ArcRectangleWidth + Rectangle.X, Rectangle.Y, ArcRectangleWidth, ArcRectangleWidth), -90, 90)
        P.AddArc(New Rectangle(Rectangle.Width - ArcRectangleWidth + Rectangle.X, Rectangle.Height - ArcRectangleWidth + Rectangle.Y, ArcRectangleWidth, ArcRectangleWidth), 0, 90)
        P.AddArc(New Rectangle(Rectangle.X, Rectangle.Height - ArcRectangleWidth + Rectangle.Y, ArcRectangleWidth, ArcRectangleWidth), 90, 90)
        P.AddLine(New Point(Rectangle.X, Rectangle.Height - ArcRectangleWidth + Rectangle.Y), New Point(Rectangle.X, Curve + Rectangle.Y))
        Return P
    End Function
    'Public Function RoundRect(ByVal X As Integer, ByVal Y As Integer, ByVal Width As Integer, ByVal Height As Integer, ByVal Curve As Integer) As GraphicsPath
    '    Dim Rectangle As Rectangle = New Rectangle(X, Y, Width, Height)
    '    Dim P As GraphicsPath = New GraphicsPath()
    '    Dim ArcRectangleWidth As Integer = Curve * 2
    '    P.AddArc(New Rectangle(Rectangle.X, Rectangle.Y, ArcRectangleWidth, ArcRectangleWidth), -180, 90)
    '    P.AddArc(New Rectangle(Rectangle.Width - ArcRectangleWidth + Rectangle.X, Rectangle.Y, ArcRectangleWidth, ArcRectangleWidth), -90, 90)
    '    P.AddArc(New Rectangle(Rectangle.Width - ArcRectangleWidth + Rectangle.X, Rectangle.Height - ArcRectangleWidth + Rectangle.Y, ArcRectangleWidth, ArcRectangleWidth), 0, 90)
    '    P.AddArc(New Rectangle(Rectangle.X, Rectangle.Height - ArcRectangleWidth + Rectangle.Y, ArcRectangleWidth, ArcRectangleWidth), 90, 90)
    '    P.AddLine(New Point(Rectangle.X, Rectangle.Height - ArcRectangleWidth + Rectangle.Y), New Point(Rectangle.X, Curve + Rectangle.Y))
    '    Return P
    'End Function
End Module
MustInherit Class ThemeControl
    Inherits Control

#Region " Initialization "

    Protected G As Graphics, B As Bitmap
    Sub New()
        SetStyle(DirectCast(139270, ControlStyles), True)
        B = New Bitmap(1, 1)
        G = Graphics.FromImage(B)
    End Sub

    Sub AllowTransparent()
        SetStyle(ControlStyles.Opaque, False)
        SetStyle(ControlStyles.SupportsTransparentBackColor, True)
    End Sub

    Overrides Property Text() As String
        Get
            Return MyBase.Text
        End Get
        Set(ByVal v As String)
            MyBase.Text = v
            Invalidate()
        End Set
    End Property
#End Region

#Region " Mouse Handling "

    Protected Enum State As Byte
        MouseNone = 0
        MouseOver = 1
        MouseDown = 2
    End Enum

    Protected MouseState As State
    Protected Overrides Sub OnMouseLeave(ByVal e As EventArgs)
        ChangeMouseState(State.MouseNone)
        MyBase.OnMouseLeave(e)
    End Sub
    Protected Overrides Sub OnMouseEnter(ByVal e As EventArgs)
        ChangeMouseState(State.MouseOver)
        MyBase.OnMouseEnter(e)
    End Sub
    Protected Overrides Sub OnMouseUp(ByVal e As MouseEventArgs)
        ChangeMouseState(State.MouseOver)
        MyBase.OnMouseUp(e)
    End Sub
    Protected Overrides Sub OnMouseDown(ByVal e As MouseEventArgs)
        If e.Button = MouseButtons.Left Then ChangeMouseState(State.MouseDown)
        MyBase.OnMouseDown(e)
    End Sub

    Private Sub ChangeMouseState(ByVal e As State)
        MouseState = e
        Invalidate()
    End Sub

#End Region

#Region " Convienence "

    MustOverride Sub PaintHook()
    Protected NotOverridable Overrides Sub OnPaint(ByVal e As PaintEventArgs)
        If Width = 0 OrElse Height = 0 Then Return
        PaintHook()
        e.Graphics.DrawImage(B, 0, 0)
    End Sub

    Protected Overrides Sub OnSizeChanged(ByVal e As EventArgs)
        If Not Width = 0 AndAlso Not Height = 0 Then
            B = New Bitmap(Width, Height)
            G = Graphics.FromImage(B)
            Invalidate()
        End If
        MyBase.OnSizeChanged(e)
    End Sub

    Private _NoRounding As Boolean
    Property NoRounding() As Boolean
        Get
            Return _NoRounding
        End Get
        Set(ByVal v As Boolean)
            _NoRounding = v
            Invalidate()
        End Set
    End Property

    Private _Image As Image
    Property Image() As Image
        Get
            Return _Image
        End Get
        Set(ByVal value As Image)
            _Image = value
            Invalidate()
        End Set
    End Property
    ReadOnly Property ImageWidth() As Integer
        Get
            If _Image Is Nothing Then Return 0
            Return _Image.Width
        End Get
    End Property
    ReadOnly Property ImageTop() As Integer
        Get
            If _Image Is Nothing Then Return 0
            Return Height \ 2 - _Image.Height \ 2
        End Get
    End Property

    Private _Size As Size
    Private _Rectangle As Rectangle
    Private _Gradient As LinearGradientBrush
    Private _Brush As SolidBrush

    Protected Sub DrawCorners(ByVal c As Color, ByVal rect As Rectangle)
        If _NoRounding Then Return

        B.SetPixel(rect.X, rect.Y, c)
        B.SetPixel(rect.X + (rect.Width - 1), rect.Y, c)
        B.SetPixel(rect.X, rect.Y + (rect.Height - 1), c)
        B.SetPixel(rect.X + (rect.Width - 1), rect.Y + (rect.Height - 1), c)
    End Sub

    Protected Sub DrawBorders(ByVal p1 As Pen, ByVal p2 As Pen, ByVal rect As Rectangle)
        G.DrawRectangle(p1, rect.X, rect.Y, rect.Width - 1, rect.Height - 1)
        G.DrawRectangle(p2, rect.X + 1, rect.Y + 1, rect.Width - 3, rect.Height - 3)
    End Sub

    Protected Sub DrawText(ByVal a As HorizontalAlignment, ByVal c As Color, ByVal x As Integer)
        DrawText(a, c, x, 0)
    End Sub
    Protected Sub DrawText(ByVal a As HorizontalAlignment, ByVal c As Color, ByVal x As Integer, ByVal y As Integer)
        If String.IsNullOrEmpty(Text) Then Return
        _Size = G.MeasureString(Text, Font).ToSize
        _Brush = New SolidBrush(c)

        Select Case a
            Case HorizontalAlignment.Left
                G.DrawString(Text, Font, _Brush, x, Height \ 2 - _Size.Height \ 2 + y)
            Case HorizontalAlignment.Right
                G.DrawString(Text, Font, _Brush, Width - _Size.Width - x, Height \ 2 - _Size.Height \ 2 + y)
            Case HorizontalAlignment.Center
                G.DrawString(Text, Font, _Brush, Width \ 2 - _Size.Width \ 2 + x, Height \ 2 - _Size.Height \ 2 + y)
        End Select
    End Sub

    Protected Sub DrawIcon(ByVal a As HorizontalAlignment, ByVal x As Integer)
        DrawIcon(a, x, 0)
    End Sub
    Protected Sub DrawIcon(ByVal a As HorizontalAlignment, ByVal x As Integer, ByVal y As Integer)
        If _Image Is Nothing Then Return
        Select Case a
            Case HorizontalAlignment.Left
                G.DrawImage(_Image, x, Height \ 2 - _Image.Height \ 2 + y)
            Case HorizontalAlignment.Right
                G.DrawImage(_Image, Width - _Image.Width - x, Height \ 2 - _Image.Height \ 2 + y)
            Case HorizontalAlignment.Center
                G.DrawImage(_Image, Width \ 2 - _Image.Width \ 2, Height \ 2 - _Image.Height \ 2)
        End Select
    End Sub

    Protected Sub DrawGradient(ByVal c1 As Color, ByVal c2 As Color, ByVal x As Integer, ByVal y As Integer, ByVal width As Integer, ByVal height As Integer, ByVal angle As Single)
        _Rectangle = New Rectangle(x, y, width, height)
        _Gradient = New LinearGradientBrush(_Rectangle, c1, c2, angle)
        G.FillRectangle(_Gradient, _Rectangle)
    End Sub
#End Region

End Class
MustInherit Class ThemeContainerControl
    Inherits ContainerControl

#Region " Initialization "

    Protected G As Graphics, B As Bitmap
    Sub New()
        SetStyle(DirectCast(139270, ControlStyles), True)
        B = New Bitmap(1, 1)
        G = Graphics.FromImage(B)
    End Sub

    Sub AllowTransparent()
        SetStyle(ControlStyles.Opaque, False)
        SetStyle(ControlStyles.SupportsTransparentBackColor, True)
    End Sub

#End Region
#Region " Convienence "

    MustOverride Sub PaintHook()
    Protected NotOverridable Overrides Sub OnPaint(ByVal e As PaintEventArgs)
        If Width = 0 OrElse Height = 0 Then Return
        PaintHook()
        e.Graphics.DrawImage(B, 0, 0)
    End Sub

    Protected Overrides Sub OnSizeChanged(ByVal e As EventArgs)
        If Not Width = 0 AndAlso Not Height = 0 Then
            B = New Bitmap(Width, Height)
            G = Graphics.FromImage(B)
            Invalidate()
        End If
        MyBase.OnSizeChanged(e)
    End Sub

    Private _NoRounding As Boolean
    Property NoRounding() As Boolean
        Get
            Return _NoRounding
        End Get
        Set(ByVal v As Boolean)
            _NoRounding = v
            Invalidate()
        End Set
    End Property

    Private _Rectangle As Rectangle
    Private _Gradient As LinearGradientBrush

    Protected Sub DrawCorners(ByVal c As Color, ByVal rect As Rectangle)
        If _NoRounding Then Return
        B.SetPixel(rect.X, rect.Y, c)
        B.SetPixel(rect.X + (rect.Width - 1), rect.Y, c)
        B.SetPixel(rect.X, rect.Y + (rect.Height - 1), c)
        B.SetPixel(rect.X + (rect.Width - 1), rect.Y + (rect.Height - 1), c)
    End Sub

    Protected Sub DrawBorders(ByVal p1 As Pen, ByVal p2 As Pen, ByVal rect As Rectangle)
        G.DrawRectangle(p1, rect.X, rect.Y, rect.Width - 1, rect.Height - 1)
        G.DrawRectangle(p2, rect.X + 1, rect.Y + 1, rect.Width - 3, rect.Height - 3)
    End Sub

    Protected Sub DrawGradient(ByVal c1 As Color, ByVal c2 As Color, ByVal x As Integer, ByVal y As Integer, ByVal width As Integer, ByVal height As Integer, ByVal angle As Single)
        _Rectangle = New Rectangle(x, y, width, height)
        _Gradient = New LinearGradientBrush(_Rectangle, c1, c2, angle)
        G.FillRectangle(_Gradient, _Rectangle)
    End Sub
#End Region

End Class

Class TxtBox
    Inherits ThemeControl
    Dim WithEvents txtbox As New TextBox
#Region "lol"
    Private _passmask As Boolean = False
    Public Shadows Property UseSystemPasswordChar() As Boolean
        Get
            Return _passmask
        End Get
        Set(ByVal v As Boolean)
            txtbox.UseSystemPasswordChar = UseSystemPasswordChar
            _passmask = v
            Invalidate()
        End Set
    End Property
    Private _maxchars As Integer = 32767
    Public Shadows Property MaxLength() As Integer
        Get
            Return _maxchars
        End Get
        Set(ByVal v As Integer)
            _maxchars = v
            txtbox.MaxLength = MaxLength
            Invalidate()
        End Set
    End Property
    Private _align As HorizontalAlignment
    Public Shadows Property TextAlignment() As HorizontalAlignment
        Get
            Return _align
        End Get
        Set(ByVal v As HorizontalAlignment)
            _align = v
            Invalidate()
        End Set
    End Property

    Protected Overrides Sub OnPaintBackground(ByVal pevent As System.Windows.Forms.PaintEventArgs)
    End Sub
    Protected Overrides Sub OnTextChanged(ByVal e As System.EventArgs)
        MyBase.OnTextChanged(e)
        Invalidate()
    End Sub
    Protected Overrides Sub OnBackColorChanged(ByVal e As System.EventArgs)
        MyBase.OnBackColorChanged(e)
        txtbox.BackColor = BackColor
        Invalidate()
    End Sub
    Protected Overrides Sub OnForeColorChanged(ByVal e As System.EventArgs)
        MyBase.OnForeColorChanged(e)
        txtbox.ForeColor = ForeColor
        Invalidate()
    End Sub
    Protected Overrides Sub OnFontChanged(ByVal e As System.EventArgs)
        MyBase.OnFontChanged(e)
        txtbox.Font = Font
    End Sub
    Protected Overrides Sub OnGotFocus(ByVal e As System.EventArgs)
        MyBase.OnGotFocus(e)
        txtbox.Focus()
    End Sub
    Sub TextChngTxtBox() Handles txtbox.TextChanged
        Text = txtbox.Text
    End Sub
    Sub TextChng() Handles MyBase.TextChanged
        txtbox.Text = Text
    End Sub

#End Region

    Protected Overrides Sub WndProc(ByRef m As Message)
        Select Case m.Msg
            Case 15
                Invalidate()
                MyBase.WndProc(m)
                Me.PaintHook()
                Exit Select
            Case Else
                MyBase.WndProc(m)
                Exit Select
        End Select
    End Sub

    Sub New()
        MyBase.New()

        Controls.Add(txtbox)
        With txtbox
            .Multiline = False
            .BackColor = Color.FromArgb(0, 0, 0)
            .ForeColor = ForeColor
            .Text = String.Empty
            .TextAlign = HorizontalAlignment.Center
            .BorderStyle = BorderStyle.None
            .Location = New Point(5, 8)
            .Font = New Font("Arial", 8.25F, FontStyle.Bold)
            .Size = New Size(Width - 8, Height - 11)
            .UseSystemPasswordChar = UseSystemPasswordChar
        End With

        Text = ""

        DoubleBuffered = True
    End Sub

    Overrides Sub PaintHook()
        Me.BackColor = Color.White
        G.Clear(Parent.BackColor)
        Dim p As New Pen(Color.FromArgb(204, 204, 204), 1)
        Dim o As New Pen(Color.FromArgb(252, 252, 252), 7)
        G.FillPath(Brushes.White, Draw.RoundRect(New Rectangle(0, 0, Width - 1, Height - 1), 2))
        G.DrawPath(o, Draw.RoundRect(New Rectangle(0, 0, Width - 1, Height - 1), 2))
        G.DrawPath(p, Draw.RoundRect(New Rectangle(0, 0, Width - 1, Height - 1), 2))
        Height = txtbox.Height + 16
        Dim drawFont As New Font("Tahoma", 9, FontStyle.Regular)
        With txtbox
            .Width = Width - 12
            .ForeColor = Color.FromArgb(72, 72, 72)
            .Font = drawFont
            .TextAlign = TextAlignment
            .UseSystemPasswordChar = UseSystemPasswordChar
        End With
        DrawCorners(Parent.BackColor, ClientRectangle)
    End Sub
End Class
Class MultiTxtBox
    Inherits ThemeControl
    Dim WithEvents txtbox As New TextBox
#Region "lol"
    Private _passmask As Boolean = False
    Public Shadows Property UseSystemPasswordChar() As Boolean
        Get
            Return _passmask
        End Get
        Set(ByVal v As Boolean)
            txtbox.UseSystemPasswordChar = UseSystemPasswordChar
            _passmask = v
            Invalidate()
        End Set
    End Property
    Private _maxchars As Integer = 32767
    Public Shadows Property MaxLength() As Integer
        Get
            Return _maxchars
        End Get
        Set(ByVal v As Integer)
            _maxchars = v
            txtbox.MaxLength = MaxLength
            Invalidate()
        End Set
    End Property
    Private _align As HorizontalAlignment
    Public Shadows Property TextAlignment() As HorizontalAlignment
        Get
            Return _align
        End Get
        Set(ByVal v As HorizontalAlignment)
            _align = v
            Invalidate()
        End Set
    End Property

    Protected Overrides Sub OnPaintBackground(ByVal pevent As System.Windows.Forms.PaintEventArgs)
    End Sub
    Protected Overrides Sub OnTextChanged(ByVal e As System.EventArgs)
        MyBase.OnTextChanged(e)
        Invalidate()
    End Sub
    Protected Overrides Sub OnBackColorChanged(ByVal e As System.EventArgs)
        MyBase.OnBackColorChanged(e)
        txtbox.BackColor = BackColor
        Invalidate()
    End Sub
    Protected Overrides Sub OnForeColorChanged(ByVal e As System.EventArgs)
        MyBase.OnForeColorChanged(e)
        txtbox.ForeColor = ForeColor
        Invalidate()
    End Sub
    Protected Overrides Sub OnFontChanged(ByVal e As System.EventArgs)
        MyBase.OnFontChanged(e)
        txtbox.Font = Font
    End Sub
    Protected Overrides Sub OnGotFocus(ByVal e As System.EventArgs)
        MyBase.OnGotFocus(e)
        txtbox.Focus()
    End Sub
    Sub TextChngTxtBox() Handles TxtBox.TextChanged
        Text = txtbox.Text
    End Sub
    Sub TextChng() Handles MyBase.TextChanged
        txtbox.Text = Text
    End Sub

#End Region

    Protected Overrides Sub WndProc(ByRef m As Message)
        Select Case m.Msg
            Case 15
                Invalidate()
                MyBase.WndProc(m)
                Me.PaintHook()
                Exit Select
            Case Else
                MyBase.WndProc(m)
                Exit Select
        End Select
    End Sub

    Sub New()
        MyBase.New()

        Controls.Add(txtbox)
        With txtbox
            .ScrollBars = ScrollBars.Vertical
            .Multiline = True
            .BackColor = Color.FromArgb(0, 0, 0)
            .ForeColor = ForeColor
            .Text = String.Empty
            .TextAlign = HorizontalAlignment.Center
            .BorderStyle = BorderStyle.None
            .Location = New Point(5, 8)
            .Font = New Font("Arial", 8.25F, FontStyle.Bold)
            .Size = New Size(Width - 8, Height - 11)
            .UseSystemPasswordChar = UseSystemPasswordChar
        End With

        Text = ""

        DoubleBuffered = True
    End Sub

    Overrides Sub PaintHook()
        Me.BackColor = Color.White
        G.Clear(Parent.BackColor)
        Dim p As New Pen(Color.FromArgb(204, 204, 204), 1)
        Dim o As New Pen(Color.FromArgb(252, 252, 252), 7)
        G.FillPath(Brushes.White, Draw.RoundRect(New Rectangle(0, 0, Width - 1, Height - 1), 2))
        G.DrawPath(o, Draw.RoundRect(New Rectangle(0, 0, Width - 1, Height - 1), 2))
        G.DrawPath(p, Draw.RoundRect(New Rectangle(0, 0, Width - 1, Height - 1), 2))
        Dim drawFont As New Font("Tahoma", 9, FontStyle.Regular)
        With txtbox
            .Size = New Size(Width - 10, Height - 14)
            .ForeColor = Color.FromArgb(72, 72, 72)
            .Font = drawFont
            .TextAlign = TextAlignment
            .UseSystemPasswordChar = UseSystemPasswordChar
        End With
        DrawCorners(Parent.BackColor, ClientRectangle)
    End Sub
End Class

Class PanelBox
    Inherits ThemeContainerControl
    Private _Checked As Boolean
    Sub New()
        AllowTransparent()
    End Sub
    Overrides Sub PaintHook()
        Me.Font = New Font("Tahoma", 10)
        Me.ForeColor = Color.FromArgb(40, 40, 40)
        G.SmoothingMode = SmoothingMode.AntiAlias
        G.FillRectangle(New SolidBrush(Color.FromArgb(235, 235, 235)), New Rectangle(2, 0, Width, Height))
        G.FillRectangle(New SolidBrush(Color.FromArgb(249, 249, 249)), New Rectangle(1, 0, Width - 3, Height - 4))
        G.DrawRectangle(New Pen(Color.FromArgb(214, 214, 214)), 0, 0, Width - 2, Height - 3)
    End Sub
End Class
Class GroupDropBox
    Inherits ThemeContainerControl
    Private _Checked As Boolean
    Private X As Integer
    Private y As Integer
    Private _OpenedSize As Size

    Public Property Checked As Boolean
        Get
            Return _Checked
        End Get
        Set(ByVal V As Boolean)
            _Checked = V
            Invalidate()
        End Set
    End Property
    Public Property OpenSize As Size
        Get
            Return _OpenedSize
        End Get
        Set(ByVal V As Size)
            _OpenedSize = V
            Invalidate()
        End Set
    End Property
    Sub New()
        AllowTransparent()
        Size = New Size(90, 30)
        MinimumSize = New Size(5, 30)
        _Checked = True
    End Sub
    Overrides Sub PaintHook()
        Me.Font = New Font("Tahoma", 10)
        Me.ForeColor = Color.FromArgb(40, 40, 40)
        If _Checked = True Then
            G.SmoothingMode = SmoothingMode.AntiAlias
            G.Clear(Color.FromArgb(245, 245, 245))
            G.FillRectangle(New SolidBrush(Color.FromArgb(231, 231, 231)), New Rectangle(0, 0, Width, 30))
            G.DrawLine(New Pen(Color.FromArgb(237, 237, 237)), 1, 1, Width - 2, 1)
            G.DrawRectangle(New Pen(Color.FromArgb(214, 214, 214)), 0, 0, Width - 1, Height - 1)
            G.DrawRectangle(New Pen(Color.FromArgb(214, 214, 214)), 0, 0, Width - 1, 30)
            Me.Size = _OpenedSize
            G.DrawString("t", New Font("Marlett", 12), New SolidBrush(Me.ForeColor), Width - 25, 5)
        Else
            G.SmoothingMode = SmoothingMode.AntiAlias
            G.Clear(Color.FromArgb(245, 245, 245))
            G.FillRectangle(New SolidBrush(Color.FromArgb(231, 231, 231)), New Rectangle(0, 0, Width, 30))
            G.DrawLine(New Pen(Color.FromArgb(237, 237, 237)), 1, 1, Width - 2, 1)
            G.DrawRectangle(New Pen(Color.FromArgb(214, 214, 214)), 0, 0, Width - 1, Height - 1)
            G.DrawRectangle(New Pen(Color.FromArgb(214, 214, 214)), 0, 0, Width - 1, 30)
            Me.Size = New Size(Width, 30)
            G.DrawString("u", New Font("Marlett", 12), New SolidBrush(Me.ForeColor), Width - 25, 5)
        End If
        G.DrawString(Text, Font, New SolidBrush(Me.ForeColor), 7, 6)
    End Sub

    Private Sub meResize(ByVal sender As Object, ByVal e As System.EventArgs) Handles Me.Resize
        If _Checked = True Then
            _OpenedSize = Me.Size
        Else
        End If
    End Sub


    Protected Overrides Sub OnMouseMove(ByVal e As System.Windows.Forms.MouseEventArgs)
        MyBase.OnMouseMove(e)
        X = e.X
        y = e.Y
        Invalidate()
    End Sub

    Sub changeCheck() Handles Me.MouseDown


        If X >= Width - 22 Then
            If y <= 30 Then
                Select Case Checked
                    Case True
                        Checked = False
                    Case False
                        Checked = True
                End Select
            End If
        End If
    End Sub
End Class
Class GroupPanelBox
    Inherits ThemeContainerControl
    Private _Checked As Boolean
    Sub New()
        AllowTransparent()
    End Sub
    Overrides Sub PaintHook()
        Me.Font = New Font("Tahoma", 10)
        Me.ForeColor = Color.FromArgb(40, 40, 40)
        G.SmoothingMode = SmoothingMode.AntiAlias
        G.Clear(Color.FromArgb(245, 245, 245))
        G.FillRectangle(New SolidBrush(Color.FromArgb(231, 231, 231)), New Rectangle(0, 0, Width, 30))
        G.DrawLine(New Pen(Color.FromArgb(233, 238, 240)), 1, 1, Width - 2, 1)
        G.DrawRectangle(New Pen(Color.FromArgb(214, 214, 214)), 0, 0, Width - 1, Height - 1)
        G.DrawRectangle(New Pen(Color.FromArgb(214, 214, 214)), 0, 0, Width - 1, 30)
        G.DrawString(Text, Font, New SolidBrush(Me.ForeColor), 7, 6)
    End Sub
End Class

Class ButtonGreen
    Inherits ThemeControl
    Overrides Sub PaintHook()
        Me.Font = New Font("Arial", 10)
        G.Clear(Me.BackColor)
        G.SmoothingMode = SmoothingMode.HighQuality
        Select Case MouseState
            Case State.MouseNone
                Dim p As New Pen(Color.FromArgb(120, 159, 22), 1)
                Dim x As New LinearGradientBrush(ClientRectangle, Color.FromArgb(157, 209, 57), Color.FromArgb(130, 181, 18), LinearGradientMode.Vertical)
                G.FillPath(x, Draw.RoundRect(ClientRectangle, 4))
                G.DrawPath(p, Draw.RoundRect(New Rectangle(0, 0, Width - 1, Height - 1), 3))
                G.DrawLine(New Pen(Color.FromArgb(190, 232, 109)), 2, 1, Width - 3, 1)
                DrawText(HorizontalAlignment.Center, Color.FromArgb(240, 240, 240), 0)
            Case State.MouseDown
                Dim p As New Pen(Color.FromArgb(120, 159, 22), 1)
                Dim x As New LinearGradientBrush(ClientRectangle, Color.FromArgb(125, 171, 25), Color.FromArgb(142, 192, 40), LinearGradientMode.Vertical)
                G.FillPath(x, Draw.RoundRect(ClientRectangle, 4))
                G.DrawPath(p, Draw.RoundRect(New Rectangle(0, 0, Width - 1, Height - 1), 3))
                G.DrawLine(New Pen(Color.FromArgb(142, 172, 30)), 2, 1, Width - 3, 1)
                DrawText(HorizontalAlignment.Center, Color.FromArgb(250, 250, 250), 1)
            Case State.MouseOver
                Dim p As New Pen(Color.FromArgb(120, 159, 22), 1)
                Dim x As New LinearGradientBrush(ClientRectangle, Color.FromArgb(165, 220, 59), Color.FromArgb(137, 191, 18), LinearGradientMode.Vertical)
                G.FillPath(x, Draw.RoundRect(ClientRectangle, 4))
                G.DrawPath(p, Draw.RoundRect(New Rectangle(0, 0, Width - 1, Height - 1), 3))
                G.DrawLine(New Pen(Color.FromArgb(190, 232, 109)), 2, 1, Width - 3, 1)
                DrawText(HorizontalAlignment.Center, Color.FromArgb(240, 240, 240), -1)
        End Select
        Me.Cursor = Cursors.Hand
    End Sub
End Class
Class ButtonBlue
    Inherits ThemeControl
    Overrides Sub PaintHook()
        Me.Font = New Font("Arial", 10)
        G.Clear(Me.BackColor)
        G.SmoothingMode = SmoothingMode.HighQuality
        Select Case MouseState
            Case State.MouseNone
                Dim p As New Pen(Color.FromArgb(34, 112, 171), 1)
                Dim x As New LinearGradientBrush(ClientRectangle, Color.FromArgb(51, 159, 231), Color.FromArgb(33, 128, 206), LinearGradientMode.Vertical)
                G.FillPath(x, Draw.RoundRect(ClientRectangle, 4))
                G.DrawPath(p, Draw.RoundRect(New Rectangle(0, 0, Width - 1, Height - 1), 3))
                G.DrawLine(New Pen(Color.FromArgb(131, 197, 241)), 2, 1, Width - 3, 1)
                DrawText(HorizontalAlignment.Center, Color.FromArgb(240, 240, 240), 0)
            Case State.MouseDown
                Dim p As New Pen(Color.FromArgb(34, 112, 171), 1)
                Dim x As New LinearGradientBrush(ClientRectangle, Color.FromArgb(37, 124, 196), Color.FromArgb(53, 153, 219), LinearGradientMode.Vertical)
                G.FillPath(x, Draw.RoundRect(ClientRectangle, 4))
                G.DrawPath(p, Draw.RoundRect(New Rectangle(0, 0, Width - 1, Height - 1), 3))

                DrawText(HorizontalAlignment.Center, Color.FromArgb(250, 250, 250), 1)
            Case State.MouseOver
                Dim p As New Pen(Color.FromArgb(34, 112, 171), 1)
                Dim x As New LinearGradientBrush(ClientRectangle, Color.FromArgb(54, 167, 243), Color.FromArgb(35, 165, 217), LinearGradientMode.Vertical)
                G.FillPath(x, Draw.RoundRect(ClientRectangle, 4))
                G.DrawPath(p, Draw.RoundRect(New Rectangle(0, 0, Width - 1, Height - 1), 3))
                G.DrawLine(New Pen(Color.FromArgb(131, 197, 241)), 2, 1, Width - 3, 1)
                DrawText(HorizontalAlignment.Center, Color.FromArgb(240, 240, 240), -1)
        End Select
        Me.Cursor = Cursors.Hand
    End Sub
End Class

Class StatusBar
    Inherits ThemeControl
    Sub New()
        Me.Dock = DockStyle.Bottom
        Me.Size = New Size(Width, 20)
    End Sub
    Overrides Sub PaintHook()
        Me.Font = New Font("Arial", 10)
        G.Clear(Me.BackColor)
        G.SmoothingMode = SmoothingMode.HighQuality

        Select Case MouseState
            Case State.MouseNone
                DrawGradient(Color.FromArgb(20, 82, 179), Color.FromArgb(58, 110, 195), 0, 0, Width, Height, 270S)
                G.DrawRectangle(New Pen(Color.FromArgb(12, 69, 180)), 0, 0, Width - 1, Height - 1)
                DrawText(HorizontalAlignment.Left, Color.FromArgb(240, 240, 240), +1)
            Case State.MouseDown
                DrawGradient(Color.FromArgb(19, 75, 172), Color.FromArgb(70, 110, 198), 0, 0, Width, Height, 270S)
                G.DrawRectangle(New Pen(Color.FromArgb(12, 69, 180)), 0, 0, Width - 1, Height - 1)
                DrawText(HorizontalAlignment.Left, Color.FromArgb(232, 232, 232), +1)
            Case State.MouseOver
                DrawGradient(Color.FromArgb(21, 79, 177), Color.FromArgb(76, 128, 218), 0, 0, Width, Height, 270S)
                G.DrawRectangle(New Pen(Color.FromArgb(12, 69, 180)), 0, 0, Width - 1, Height - 1)
                DrawText(HorizontalAlignment.Left, Color.FromArgb(250, 250, 250), +1)
        End Select
        G.DrawLine(New Pen(Color.FromArgb(50, 255, 255, 255)), 1, 1, Width - 3, 1)

    End Sub
End Class
Class ProgressBar
    Inherits ThemeControl
    Private _Maximum As Integer
    Dim Gloss As Boolean
    Dim Vertical As Boolean = True
    Public Property VerticalAlignment As Boolean
        Get
            Return Vertical
        End Get
        Set(ByVal v As Boolean)
            Vertical = v
            Invalidate()
        End Set
    End Property
    Public Property Glossy As Boolean
        Get
            Return Gloss
        End Get
        Set(ByVal v As Boolean)
            Gloss = v
            Invalidate()
        End Set
    End Property
    Public Property Maximum() As Integer
        Get
            Return _Maximum
        End Get
        Set(ByVal v As Integer)
            Select Case v
                Case Is < _Value
                    _Value = v
            End Select
            _Maximum = v
            Invalidate()
        End Set
    End Property
    Private _Value As Integer
    Public Property Value() As Integer
        Get
            Return _Value
        End Get
        Set(ByVal v As Integer)
            Select Case v
                Case Is > _Maximum
                    v = _Maximum
            End Select
            _Value = v
            Invalidate()
        End Set
    End Property
    Overrides Sub PaintHook()
        G.SmoothingMode = SmoothingMode.HighQuality
        G.Clear(Color.Transparent)
        Dim s As Integer = 0
        Dim pe As New Pen(Color.FromArgb(34, 112, 171), 1)
        Dim xe As New LinearGradientBrush(ClientRectangle, Color.FromArgb(31, 119, 181), Color.FromArgb(33, 128, 206), LinearGradientMode.Vertical)
        G.FillPath(xe, Draw.RoundRect(ClientRectangle, 4))
        G.DrawPath(pe, Draw.RoundRect(New Rectangle(0, 0, Width - 1, Height - 1), 3))
        G.DrawLine(New Pen(Color.FromArgb(65, 131, 197, 241)), 2, 1, Width - 3, 1)

        'Fill
        If _Value > 1 Then
            If Vertical Then
                s = (Height - CInt(_Value / _Maximum * Height))
                If Glossy Then

                    Dim p As New Pen(Color.FromArgb(34, 112, 171), 1)
                    Dim x As New LinearGradientBrush(ClientRectangle, Color.FromArgb(51, 159, 231), Color.FromArgb(33, 128, 206), LinearGradientMode.Vertical)
                    G.FillPath(x, Draw.RoundRect(New Rectangle(2, s + 3, Width - 5, CInt(_Value / _Maximum * Height) - 6), 4))
                    DrawGradient(Color.FromArgb(50, Color.White), Color.Transparent, 4, s + 3, 10, CInt(_Value / _Maximum * Height) - 7, 270S)
                    G.DrawPath(p, Draw.RoundRect(New Rectangle(2, s + 3, Width - 5, CInt(_Value / _Maximum * Height) - 6), 3))
                    G.DrawLine(New Pen(Color.FromArgb(90, 131, 197, 241)), 4, s + 4, Width - 6, s + 4)
                ElseIf Not Glossy Then
                    Dim p As New Pen(Color.FromArgb(34, 112, 171), 1)
                    Dim x As New LinearGradientBrush(ClientRectangle, Color.FromArgb(51, 159, 231), Color.FromArgb(33, 128, 206), LinearGradientMode.Vertical)
                    G.FillPath(x, Draw.RoundRect(New Rectangle(2, s + 3, Width - 5, CInt(_Value / _Maximum * Height) - 6), 4))
                    G.DrawPath(p, Draw.RoundRect(New Rectangle(2, s + 3, Width - 5, CInt(_Value / _Maximum * Height) - 6), 3))
                    G.DrawLine(New Pen(Color.FromArgb(90, 131, 197, 241)), 4, s + 4, Width - 6, s + 4)

                End If
            ElseIf Not Vertical Then
                s = (Height - CInt(_Value / _Maximum * Width))
                If Glossy Then

                    Dim p As New Pen(Color.FromArgb(34, 112, 171), 1)
                    Dim x As New LinearGradientBrush(ClientRectangle, Color.FromArgb(51, 159, 231), Color.FromArgb(33, 128, 206), LinearGradientMode.Vertical)
                    G.FillPath(x, Draw.RoundRect(New Rectangle(2, 2, CInt(_Value / _Maximum * Width) - 6, Height - 5), 4))
                    G.DrawLine(New Pen(Color.FromArgb(90, 131, 197, 241)), 4, 3, CInt(_Value / _Maximum * Width) - 7, 3)
                    DrawGradient(Color.FromArgb(60, Color.White), Color.Transparent, 3, 3, CInt(_Value / _Maximum * Width) - 7, Height / 2 - 3, 0S)
                    G.DrawPath(p, Draw.RoundRect(New Rectangle(2, 2, CInt(_Value / _Maximum * Width) - 6, Height - 5), 3))
                ElseIf Not Glossy Then


                    Dim p As New Pen(Color.FromArgb(34, 112, 171), 1)
                    Dim x As New LinearGradientBrush(ClientRectangle, Color.FromArgb(51, 159, 231), Color.FromArgb(33, 128, 206), LinearGradientMode.Vertical)
                    G.FillPath(x, Draw.RoundRect(New Rectangle(2, 2, CInt(_Value / _Maximum * Width) - 6, Height - 5), 4))
                    G.DrawLine(New Pen(Color.FromArgb(90, 131, 197, 241)), 4, 3, CInt(_Value / _Maximum * Width) - 7, 3)
                    G.DrawPath(p, Draw.RoundRect(New Rectangle(2, 2, CInt(_Value / _Maximum * Width) - 6, Height - 5), 3))
                End If
            End If
        End If

        'Borders

    End Sub
    Public Sub Increment(ByVal Amount As Integer)
        If Me.Value + Amount > Maximum Then
            Me.Value = Maximum
        Else
            Me.Value += Amount
        End If
    End Sub

    Public Sub New()
        Me.Value = 0
        Me.Maximum = 100
        AllowTransparent()
    End Sub
End Class
Class DropDownComboBox
    Inherits ComboBox
    Private X As Integer
    Private Over As Boolean

    Sub New()
        MyBase.New()
        Font = New Font("Tahoma", 9, FontStyle.Regular)
        SetStyle(ControlStyles.AllPaintingInWmPaint Or ControlStyles.ResizeRedraw Or ControlStyles.UserPaint Or ControlStyles.DoubleBuffer, True)
        DrawMode = Windows.Forms.DrawMode.OwnerDrawFixed
        ItemHeight = 25
        DropDownStyle = ComboBoxStyle.DropDownList
    End Sub

    Protected Overrides Sub OnMouseMove(ByVal e As System.Windows.Forms.MouseEventArgs)
        MyBase.OnMouseMove(e)
        X = e.Location.X
        Invalidate()
    End Sub

    Protected Overrides Sub OnMouseEnter(ByVal e As System.EventArgs)
        MyBase.OnMouseEnter(e)
        Over = True
        Invalidate()
    End Sub

    Protected Overrides Sub OnMouseLeave(ByVal e As System.EventArgs)
        MyBase.OnMouseEnter(e)
        Over = False
        Invalidate()
    End Sub

    Protected Overrides Sub OnPaint(ByVal e As PaintEventArgs)
        Me.Font = New Font("Tahoma", 9, FontStyle.Regular)
        Dim bs As New SolidBrush(Me.ForeColor)
        If Not DropDownStyle = ComboBoxStyle.DropDownList Then DropDownStyle = ComboBoxStyle.DropDownList
        Dim B As New Bitmap(Width, Height)
        Dim G As Graphics = Graphics.FromImage(B)
        Dim m As New Font("Marlett", 11)
        G.Clear(Color.FromArgb(50, 50, 50))
        Dim GradientBrush As LinearGradientBrush = New LinearGradientBrush(New Rectangle(0, 0, Width, Height), Color.FromArgb(234, 234, 234), Color.FromArgb(242, 242, 242), 270.0F)
        G.FillRectangle(GradientBrush, New Rectangle(0, 0, Width, Height))


        Dim op As New Pen(Color.FromArgb(204, 204, 204), 1)
        Dim o As New Pen(Color.FromArgb(237, 237, 237), 6)

        G.DrawPath(o, Draw.RoundRect(New Rectangle(0, 0, Width - 1, Height - 1), 2))
        G.DrawPath(op, Draw.RoundRect(New Rectangle(0, 0, Width - 1, Height - 1), 2))

        If X >= Width - 20 And Over Then
            GradientBrush = New LinearGradientBrush(New Rectangle(0, 0, Width, Height), Color.FromArgb(239, 239, 239), Color.FromArgb(236, 236, 236), 90.0F)
            G.FillRectangle(GradientBrush, New Rectangle(Width - 22, 2, 20, Height - 4))
        ElseIf X < Width - 20 And Over Then
            GradientBrush = New LinearGradientBrush(New Rectangle(0, 0, Width, Height), Color.FromArgb(239, 239, 239), Color.FromArgb(236, 236, 236), 90.0F)
            G.FillRectangle(GradientBrush, New Rectangle(2, 2, Width - 27, Height - 4))
        End If

        Dim S1 As Integer = G.MeasureString(" ... ", Font).Height
        If SelectedIndex <> -1 Then
            G.DrawString(Items(SelectedIndex), Font, bs, 4, (Height \ 2 - S1 \ 2))
            G.DrawString("6", m, bs, Width - 22, (Height \ 2 - S1 \ 2))
        Else
            If Not Items Is Nothing And Items.Count > 0 Then
                G.DrawString(Items(0), Font, bs, 4, (Height \ 2 - S1 \ 2))
                G.DrawString("6", m, bs, Width - 22, (Height \ 2 - S1 \ 2))
            Else
                G.DrawString(" ... ", Font, bs, 4, (Height \ 2 - S1 \ 2))
                G.DrawString("6", m, bs, Width - 22, (Height \ 2 - S1 \ 2))
            End If
        End If
        G.DrawLine(New Pen(Color.FromArgb(120, 255, 255, 255)), 1, 1, Width - 3, 1)
        e.Graphics.DrawImage(B.Clone, 0, 0)

        G.Dispose() : B.Dispose()

    End Sub

    Protected Overrides Sub OnDrawItem(ByVal e As DrawItemEventArgs)
        If e.Index < 0 Then Exit Sub
        Dim rect As New Rectangle()
        rect.X = e.Bounds.X
        rect.Y = e.Bounds.Y
        rect.Width = e.Bounds.Width - 1
        rect.Height = e.Bounds.Height - 1

        e.DrawBackground()
        If e.State = 785 Or e.State = 17 Then
            e.Graphics.FillRectangle(New SolidBrush(Color.FromArgb(235, 235, 235)), e.Bounds)
            e.Graphics.FillRectangle(New SolidBrush(Color.FromArgb(50, Me.ForeColor)), e.Bounds)
            e.Graphics.DrawString(Me.Items(e.Index).ToString(), e.Font, Brushes.Black, e.Bounds.X, e.Bounds.Y + 5)
        Else
            e.Graphics.FillRectangle(New SolidBrush(Color.White), e.Bounds)
            e.Graphics.DrawString(Me.Items(e.Index).ToString(), e.Font, Brushes.Black, e.Bounds.X, e.Bounds.Y + 4)
        End If
        MyBase.OnDrawItem(e)
    End Sub

    Private Sub GhostComboBox_DropDown(ByVal sender As Object, ByVal e As System.EventArgs) Handles Me.DropDown

    End Sub

    Private Sub GhostComboBox_DropDownClosed(ByVal sender As Object, ByVal e As System.EventArgs) Handles Me.DropDownClosed
        DropDownStyle = ComboBoxStyle.Simple
        Application.DoEvents()
        DropDownStyle = ComboBoxStyle.DropDownList
    End Sub

    Private Sub GhostCombo_TextChanged(ByVal sender As Object, ByVal e As System.EventArgs) Handles Me.TextChanged
        Invalidate()
    End Sub
End Class