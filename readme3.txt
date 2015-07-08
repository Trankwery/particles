	Autor: Tomasz Jakubczyk
Plan dodania obs�ugi wielu rodzaj�w cz�stek na raz


	Koncepcja:

�eby oszcz�dza� wyko�ystywan� pami�� karty graficznej zauwa�am, �e ca�e float4 jest �adowane do pamieci.
4 pozycja zawsze zawiera 1, kt�re jest potrzebne do transformacji GL w postaci znormalizowanej.
Oryginalnie kopiowana jest ca�a tablica float4 do karty graficznej mimo, �e co 4 float jest tam zb�dny,
jest to spowodowane tym, �e kod kopiowania mi�dzy pami�ciami jest prostszy i nie trzeba wykonywa� dodatkowych
operacji zsuwania i rozsuwania pami�ci.
W efekcie w pami�ci karty s� niewyko�ystywane pozycje zawieraj�ce 1.
Chc� to wyko�ysta�;
ponumeruj� rodzaje cz�stek i przed skopiowaniem do karty b�d� przypisyw� na 4 pozycj� ka�dej cz�stce
numer jej rodzaju, a przy wczytywaniu do g��wnej pami�ci RAM dla GL'a b�d� te numery zapisywa�
do osobnej tablicy i przed wykonywaniem przekszta�ce� GL 4 pozycje zape�nie 1.


	Modyfikacje funkcji:

Plik particles_kernel_impl.cuh

operator() integrate_functor
Informacja o rodzaju cz�stki mo�e by� przechowywana w float4 posData
w metodzie b�dzie trzeba uwzgl�dni�, �e cz�stki maj� r�ne promienie (params.particleRadius).

collideSpheres
Argumenty funkcji float3 nale�y zamieni� na float4 poniewa� w tej funkcji obliczane s� si�y dzia�aj�ce
na cz�stki podczas zderzenia.

collideCell
Argument pos funkcji zmieni� z float3 na float4.
Potrzebne s� informacje o rodzaju obu cz�stek i nale�y je uwzgl�dni�; szczeg�lnie r�ne promienie.
Odpowiednio zmodyfikowa� wywo�anie collideSpheres.

collideD
Odpowiednio zmodyfikowa� wywo�anie collideCell.
Uwzgl�dni� mas� cz�stki zale�n� od jej rodzaju przy wyliczaniu nowej pr�dko�ci.

particles_kernel.cuh

SimParams
Zmieni� parametry cz�stek float (particleRadius, particleMass) na tablice float i odwo�ywa� si� do nich
przez numer typu cz�stki.

particleSystem.cpp

Chyba gdzie� w tym pliku trzeba zadba� o zamiany 1 z numerem typu cz�stki.

ParticleSystem::ParticleSystem
Ustali� parametry cz�stek dla r�nych typ�w.
Dobrze by by�o wcze�niej mie� vector z parametrami typ�w cz�stek.

ParticleSystem::initGrid, ParticleSystem::reset
Losowa� typ cz�stki z dost�pnych.
Powinna tu wkroczy� tablica z przypisanymi typami do cz�stek.

ParticleSystem::particleType
Ustali�, czy to dobry pomys� i zintegrowa� z reszt�.

ParticleSystem::update
Tu jest po��czenie sterowania CUDA i GL.
Nale�y zadba�, �eby prze��czanie 1 do normalizacji z numerem typu cz�stki przebiega�o w w�a�ciwej kolejno�ci.

particles.cpp

Zadba� o ustalenie na pocz�tku symulacji ile b�dzie typ�w cz�stek i jakie one b�d�.

